# Claude Screenshot Uploader 📸

If you run Claude Code on a remote server over SSH, you can't paste a local screenshot directly into it. This tool watches your Mac's Screenshots folder and automatically uploads new screenshots to your server, then copies the remote path to your clipboard so you can paste it straight into Claude Code.

## How it works

1. You take a screenshot on your Mac.
2. A background service notices the new file and `rsync`s it to your server over SSH.
3. The remote file path is copied to your clipboard.
4. You paste that path into Claude Code (running on the server) to reference the image.

## Requirements

- macOS
- Homebrew
- SSH key access to your server (no password prompts)

## Setup

### 1. Clone and install

```bash
git clone https://github.com/aengusl/claude-screenshot-uploader.git
cd claude-screenshot-uploader
./setup.sh
```

The script installs `fswatch`, asks for your server's hostname/user/upload path, writes `~/.claude-screenshot-uploader.conf`, sets your Mac's screenshot save folder to `~/Screenshots`, and installs a `launchd` service that runs automatically in the background (including after a reboot).

If your server has an entry in `~/.ssh/config`, use that host alias (e.g. `SERVER_HOST="my-server"`) instead of a raw IP — it picks up the right SSH key automatically.

### 2. Pick a screenshot shortcut

This tool triggers on the **file** macOS saves when you use "Save Picture of Selected Area as a File" — not the clipboard-only capture shortcuts. Check what's bound to what in **System Settings → Keyboard → Keyboard Shortcuts → Screenshots**.

If `Cmd+Shift+4` already does this on your machine, you're done. If you use that combo for something else (e.g. Raycast has claimed it for clipboard-only capture), bind "Save Picture of Selected Area as a File" to a free shortcut instead — `Cmd+Shift+1` works well and doesn't collide with anything by default.

### 3. Test it

Take a screenshot with your chosen shortcut. Within a couple seconds you should get a "Screenshot Uploaded" notification, and the remote path will be on your clipboard, ready to paste into Claude Code.

## Configuration

`~/.claude-screenshot-uploader.conf`:

```bash
SERVER_HOST="my-server"                # hostname, IP, or ~/.ssh/config alias
SERVER_USER="username"
SERVER_PATH="/tmp/screenshots"         # remote upload directory
LOCAL_SCREENSHOTS="$HOME/Screenshots"  # local folder being watched
AUTO_DELETE="false"                    # delete local file after upload?
```

## Service control

```bash
# Check it's running
launchctl list | grep screenshot

# Restart after a config or script change
launchctl unload ~/Library/LaunchAgents/com.claudecode.screenshot-uploader.plist
launchctl load ~/Library/LaunchAgents/com.claudecode.screenshot-uploader.plist

# Logs
tail -f /tmp/screenshot-uploader.log
tail -f /tmp/screenshot-uploader-error.log
```

## Troubleshooting

**Screenshots aren't uploading**
- `launchctl list | grep screenshot` — is the service running?
- `tail -30 /tmp/screenshot-uploader-error.log` — any errors?
- `ssh <user>@<host> "echo ok"` — does passwordless SSH work?
- Check the filename actually matches what's expected: the watcher looks for macOS's default screenshot names, `Screenshot ....png` or `Screen Shot ....png`. A third-party screenshot app with different naming won't be picked up.

**Nothing happens when I press my shortcut**
- Confirm the shortcut is bound to "Save Picture of Selected Area as a File" (produces a file), not "Copy Picture of Selected Area to the Clipboard" (clipboard only, no file — the watcher can't see it).

## Uninstall

```bash
./uninstall.sh
```

## License

MIT — see [LICENSE](LICENSE).
