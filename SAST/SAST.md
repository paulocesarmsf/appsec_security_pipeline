# SAST Findings Analysis

This document summarizes the three most critical vulnerabilities identified during the SAST analisys of the **FlaskBB project**. The findings were carefully reviewed in their context to better understand the potential impact and exploitation scenarios. Below you will find a detailed explanation of each vulnerability:

## Summary

| Vulnerability                  | File                          | Line | CWE   | Severity |
|--------------------------------|-------------------------------|------|-------|----------|
| Use of `eval()` for code execution | flaskbb/cli/main.py          | 382  | CWE-94 / CWE-95 | High |
|  Use of `\|safe` filter and unescaped template (possible XSS) | templates/_macros/form.html | 12   | CWE-79 | Medium |
| Template variable in `href` attribute (possible XSS) | templates/layout.html        | 54   | CWE-79 | Info |


## Finding 1 – Use of `eval()` for code execution

**Description:**  
The use of `eval()` allows dynamic execution of arbitrary code. When applied to external input, this can result in Remote `Code Execution (RCE)`, giving an attacker the ability to run commands with the same privileges as the application.

**Severity:**  
`High`. `Remote Code Execution (RCE)` vulnerabilities are generally treated as **CRITICAL** due to their impact. However, in this case the context is important: the function is part of a developer-oriented feature and exploitation requires control over the `PYTHONSTARTUP` variable or the referenced file. **This lowers the likelihood in normal local usage**, but the risk becomes **CRITICAL** if the feature is enabled in production or automated pipelines where environment variables and files may be easier to manipulate. In the next session I will explain the context of vulnerability in more detail.

**Exploitation scenario:**  
To fully understand the exploitation scenario, it was necessary to analyze the source code and evaluate the real impact given the context, that is, the function contains the `eval()`:

```python
@flaskbb.command("shell", short_help="Runs a shell in the app context.")
@with_appcontext
def shell_command():
    """Runs an interactive Python shell in the context of a given
    Flask application.  The application will populate the default
    namespace of this shell according to it"s configuration.
    This is useful for executing small snippets of management code
    without having to manually configuring the application.

    This code snippet is taken from Flask"s cli module and modified to
    run IPython and falls back to the normal shell if IPython is not
    available.
    """
    import code

    banner = "Python %s on %s\nInstance Path: %s" % (
        sys.version,
        sys.platform,
        current_app.instance_path,
    )
    ctx = {"db": db}

    # Support the regular Python interpreter startup script if someone
    # is using it.
    startup = os.environ.get("PYTHONSTARTUP")
    if startup and os.path.isfile(startup):
        with open(startup, "r") as f:
            eval(compile(f.read(), startup, "exec"), ctx)

    ctx.update(current_app.make_shell_context())

    try:
        import IPython
        from traitlets.config import get_config

        c = get_config()
        # This makes the prompt to use colors again
        c.InteractiveShellEmbed.colors = "Linux"
        IPython.embed(config=c, banner1=banner, user_ns=ctx)
    except ImportError:
        code.interact(banner=banner, local=ctx)
```
The `shell_command()` function implements the `flaskbb shell` command, which opens an **interactive shell** in the application context with access to objects like `db`. Before starting the session, it reads the `PYTHONSTARTUP` environment variable, a Python feature for loading startup scripts, and runs the referenced file using `eval(compile(..., "exec"), ctx)`. This is handy for developers locally, but it is risky because it allows arbitrary code execution from any file defined by that variable.  
If used in production or in CI/CD pipelines, an attacker who changes the `PYTHONSTARTUP` value or replaces the file it points to could run malicious code. As soon as flaskbb shell is executed, the payload would run automatically with full access to the app context, including the `db` object, making it possible to extract or modify sensitive data. This can lead to `Remote Code Execution (RCE)`, which could compromise the server, escalate privileges, or run OS commands. The risk is especially critical in automated build or deploy scenarios, since a manipulated variable could go unnoticed and impact the whole supply chain.

**Mitigation recommendation:**
- The `flaskbb shell` command should not be enabled by default. It must be controlled by a configuration flag (e.g., `ENABLE_SHELL_COMMAND=true`). If the flag is not set, the command should be disabled, ensuring it cannot be used in production or pipelines while still allowing developers to enable it locally.
- Add documentation making clear that this is a development-only feature and must not be used in production or in build/deploy pipelines.
- Remove the use of `eval()` for external files. If initialization is needed **(which I doubt)**, replace it with safer options like predefined configuration or a whitelist of allowed actions.  

### Finding 2 – Use of `|safe` filter and unescaped templates

**Description:**  
The `|safe` filter disables autoescaping in Jinja2, rendering raw HTML. If combined with user input, this can result in XSS vulnerabilities.

**Severity:**  
`Medium`. Rendering untrusted data with `|safe` allows attackers to inject arbitrary HTML/JS. In the current context no dynamic use of `description` was identified, values are static, so exploitation is not possible. However, if user-controlled input is ever passed to `description`, the severity would immediately escalate to **High** or **Critical** due to trivial exploitation.

**Exploitation scenario:**
Semgrep flagged `|safe` usage in files like `templates/_macros/form.html`. If any of these variables come from user data (e.g., post content), an attacker could embed a `<script>` tag that runs when the page loads.

**Mitigation recommendations:**  
- Avoid using `|safe` unless the data source is fully trusted.  
- Sanitize user-generated content before rendering.  
- Keep autoescaping enabled by default for all templates.  

### Finding 3 – Template variable in `href` attribute (Possible XSS)

**Description:**  
The scanner flagged template variables used inside `href` attributes as a possible XSS vector. In general, interpolating unescaped variables in `href` can allow attackers to inject `javascript:` or other malicious schemes. However, after manual review of `templates/layout.html` and `docs/_templates/layout.html`, all `href` values are either generated via safe helpers like `url_for()` or `pathto()`, or are static links. None of them are influenced by user-controlled input.

**Severity:**  
Informational. This is a false positive for the current context, since the `href` values are either static or generated through Flask’s `url_for()` helper, which safely builds URLs and cannot be manipulated by end users. If in the future a user-controlled variable is directly interpolated into an `href`, the severity would escalate to High due to trivial XSS exploitation.


**Exploitation scenario:**  
Not applicable in the current codebase, as no exploitable path was identified. The flagged instances are safe because they use framework helpers or static values.

**Mitigation recommendations:**  
- No action required for the current code.  
- As a best practice, continue using safe URL builders (`url_for`, `pathto`) instead of raw variable interpolation.  
- Document the risk so that if user-controlled data is ever placed in an `href`, it must be validated and escaped.  