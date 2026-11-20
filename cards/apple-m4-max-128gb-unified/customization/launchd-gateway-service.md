# Gateway launchd service (macOS, deployable template)

The repo plist is a TEMPLATE - `@TOKENS@` are substituted by the provisioning
script at install time, because launchd does not expand env vars or `~`.
Rendered result lives at `~/Library/LaunchAgents/<label>.plist`.

```xml
<?xml version="1.0" encoding="UTF-8"?>
<!DOCTYPE plist PUBLIC "-//Apple//DTD PLIST 1.0//EN" "http://www.apple.com/DTDs/PropertyList-1.0.dtd">
<!--
    LaunchAgent template for llama-swap.
    Tokens (@LABEL@, @BIN@, ...) are substituted by the provision script.
    KeepAlive with SuccessfulExit=false is the macOS analog of systemd's
    Restart=on-failure: restart on crash, leave alone on a clean exit.
-->
<plist version="1.0">
<dict>
    <key>Label</key>
    <string>@LABEL@</string>

    <key>ProgramArguments</key>
    <array>
        <string>@BIN@</string>
        <string>--config</string>
        <string>@CONFIG@</string>
        <string>--listen</string>
        <string>@LISTEN@</string>
    </array>

    <key>WorkingDirectory</key>
    <string>@WORKDIR@</string>

    <key>RunAtLoad</key>
    <true/>

    <key>KeepAlive</key>
    <dict>
        <key>SuccessfulExit</key>
        <false/>
    </dict>

    <key>StandardOutPath</key>
    <string>@OUTLOG@</string>

    <key>StandardErrorPath</key>
    <string>@ERRLOG@</string>

    <key>ProcessType</key>
    <string>Interactive</string>
</dict>
</plist>
```

- Verify pattern: md5 the RENDERED file, not the template.
- A startup hook preloads the resident MLX entry (zero cold start for the
  laptop's default model).
- Gateway listens on `127.0.0.1:9567`.
