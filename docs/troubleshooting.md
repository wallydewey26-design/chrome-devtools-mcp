# Troubleshooting

## General tips

- Run `npx chrome-devtools-mcp@latest --help` to test if the MCP server runs on your machine.
- Make sure that your MCP client uses the same npm and node version as your terminal.
- When configuring your MCP client, try using the `--yes` argument to `npx` to
  auto-accept installation prompt.
- Find a specific error in the output of the `chrome-devtools-mcp` server.
  Usually, if your client is an IDE, logs would be in the Output pane.

## Debugging

Start the MCP server with debugging enabled and a log file:

- `DEBUG=* npx chrome-devtools-mcp@latest --log-file=/path/to/chrome-devtools-mcp.log`

Using `.mcp.json` to debug while using a client:

```json
{
  "mcpServers": {
    "chrome-devtools": {
      "type": "stdio",
      "command": "npx",
      "args": [
        "chrome-devtools-mcp@latest",
        "--log-file",
        "/path/to/chrome-devtools-mcp.log"
      ],
      "env": {
        "DEBUG": "*"
      }
    }
  }
}
```

## Specific problems

### `Error [ERR_MODULE_NOT_FOUND]: Cannot find module ...`

This usually indicates either a non-supported Node version is in use or that the
`npm`/`npx` cache is corrupted. Try clearing the cache, uninstalling
`chrome-devtools-mcp` and installing it again. Clear the cache by running:

```sh
rm -rf ~/.npm/_npx # NOTE: this might remove other installed npx executables.
npm cache clean --force
```

### `Target closed` error

This indicates that the browser could not be started. Make sure that no Chrome
instances are running or close them. Make sure you have the latest stable Chrome
installed and that [your system is able to run Chrome](https://support.google.com/chrome/a/answer/7100626?hl=en).

### Remote debugging between virtual machine (VM) and host fails

When connecting DevTools inside a VM to Chrome running on the host, any domain is rejected by Chrome because of host header validation. Tunneling the port over SSH bypasses this restriction. In the VM, run:

```sh
ssh -N -L 127.0.0.1:9222:127.0.0.1:9222 <user>@<host-ip>
```

Point the MCP connection inside the VM to `http://127.0.0.1:9222` and DevTools
will reach the host browser without triggering the Host validation.

### Zsh-specific issues

If you're using Zsh and experiencing problems with the MCP server, consider the following:

#### Node/npm PATH not available in Zsh

Zsh loads different configuration files depending on whether the shell is interactive or non-interactive. MCP clients may start the server in a non-interactive shell, which skips some configuration files.

**Solution:**

Ensure your `PATH` and Node.js environment are configured in the correct Zsh configuration file:

- **`~/.zshenv`**: Always sourced, use this for `PATH` and environment variables needed by non-interactive shells
- **`~/.zshrc`**: Only sourced in interactive shells (terminal sessions)
- **`~/.zprofile`**: Sourced in login shells

Move your Node.js/npm PATH configuration to `~/.zshenv`:

```sh
# In ~/.zshenv
export PATH="$HOME/.nvm/versions/node/v20.19.0/bin:$PATH"
# or for nvm users:
export NVM_DIR="$HOME/.nvm"
[ -s "$NVM_DIR/nvm.sh" ] && \. "$NVM_DIR/nvm.sh"
```

After making changes, restart your MCP client or terminal session.

#### Zsh completion system interference

Zsh's completion system can sometimes interfere with command execution if misconfigured.

**Solution:**

- Ensure `compinit` is called only once in your Zsh configuration
- Verify that the `fpath` variable doesn't contain invalid or inaccessible directories
- If using Oh-My-Zsh or similar frameworks, ensure plugins don't conflict with Node.js execution

To check if completion is causing issues, temporarily disable it by commenting out the completion initialization in your `~/.zshrc`:

```sh
# autoload -Uz compinit && compinit
```

#### Environment variable conflicts

Some Zsh configurations or plugins may set environment variables that interfere with Node.js or npm execution.

**Solution:**

Check for conflicting environment variables by running:

```sh
echo $NODE_OPTIONS
echo $NPM_CONFIG_PREFIX
```

If you see unexpected values, track them down in your Zsh configuration files and adjust as needed.
