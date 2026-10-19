# Windows setup and staff pilot notes

Fork: [alj333/solidcraft](https://github.com/alj333/solidcraft). Updated 19 October 2026.

## Current status

The first Windows development PC runs SolidCraft 0.16.2 from source. The desktop
app and matching CLI bridge have been built and launched. A local SpaceMouse fix
was tested with a directly connected SpaceMouse Wireless and a Universal Receiver
both present. The user confirmed pan, rotate and zoom; 12 SpaceMouse tests and
desktop Clippy checks passed. The CLI bridge also initialized MCP, listed 15 tools
and inspected the running app's design successfully.

These checks cover one PC, not a staff rollout. At the time these notes were written,
the custom SpaceMouse change and `Start-Dev.ps1` were uncommitted on the local
`alj333-dev` branch. They are not included in this documentation change or in the
fork's `main` branch. An upstream release download does not include that local fix.
Full workspace CI and testing on other staff PCs remain outstanding.

## Developer setup on another Windows PC

Install Git, Rust through rustup, and Visual Studio 2022 Build Tools with the
**Desktop development with C++** workload and Windows SDK. The current source
requires Rust 1.95 or newer. Use PowerShell 7 for the commands below. Node.js/npm
is needed only for agents distributed through npm, not to run the CAD app.

Clone the fork into the chosen development folder:

```powershell
git clone https://github.com/alj333/solidcraft.git C:\Github\Solidcraft
Set-Location C:\Github\Solidcraft
git remote add upstream https://github.com/tangency-labs/solidcraft.git
```

Before testing custom behavior, check out the exact reviewed commit containing
the intended changes. Until those changes are published, this clone supplies the
fork's existing source only. Record `git rev-parse HEAD` with every test result.

Build both programs from the same checkout and profile:

```powershell
$env:CARGO_TARGET_DIR = Join-Path (Get-Location) 'target'
$env:CARGO_PROFILE_DEV_DEBUG_ASSERTIONS = 'false'
$env:RUST_BACKTRACE = '1'
cargo build --locked -p solidcraft -p solidcraft-cli
if ($LASTEXITCODE -ne 0) { throw 'Build failed' }
& .\target\debug\solidcraft-cli.exe --version
& .\target\debug\solidcraft.exe --sample
```

The debug-assertion setting follows the project's CI choice: some third-party
geometry assertions reject valid geometry. This retains development debug symbols
and incremental builds. The first build can take several minutes. Close the dev
app before rebuilding, then rebuild and restart after edits; Rust changes do not
hot reload. The local `Start-Dev.ps1` automates these steps once it is published.

Keep `solidcraft.exe` and `solidcraft-cli.exe` in the same folder. The AI panel
starts the CLI as an MCP bridge to the open design. Building only the desktop
package causes the "solidcraft-cli was not found next to the app" warning.
After adding the CLI, use **Stop**, then **Connect** in the AI panel.

## AI agent connectors

Install the adapter for the agent each tester will use. For example:

```powershell
npm.cmd install -g @agentclientprotocol/codex-acp
codex-acp.cmd --version
# Optional alternative:
npm.cmd install -g @agentclientprotocol/claude-agent-acp
```

Using `npm.cmd` avoids invoking PowerShell's npm script shim. In SolidCraft, open
**AI → Refresh**, select the agent, then **Connect** and follow its sign-in flow.
**Preferences → AI → Agents** provides Test and Default controls. SolidCraft
launches the adapter; a separate connector terminal need not stay open. Each staff
member signs in with their own account. Agent installation and sign-in are separate
from verifying that the agent can reach the CAD design.

See [Agent chat](agent-chat.md) for the other adapters and authentication details.

## SpaceMouse checks

On the first PC, Windows detected a SpaceMouse Wireless and a Universal Receiver,
with 3DxWare running. The original reader selected the first HID multi-axis
controller and stayed on the idle receiver. Only driver-emulated zoom worked.
The local change polls all available Windows controllers and keeps their readings
and stale-input handling separate.

In **Preferences → Navigation**, enable SpaceMouse navigation and check overall
speed, per-axis speed and inversion. On each test PC, check pan, zoom and rotation,
then release the cap and confirm motion stops. Repeat with USB, wireless, and both
receiver and USB present where available. Check unplug/reconnect and sketch-mode
pan/zoom. Record the connection type, device model and driver version. Button
mapping and every hardware model have not been validated by the initial test.

## Staff distribution plan

Staff testers should receive one portable test bundle per reviewed commit, rather
than installing compilers on every PC. Before distributing it:

1. Commit and review the intended custom changes through the fork's PR process,
   run the repository's required validation, and record the commit and results.
2. Build matching desktop and CLI binaries from that commit. Use the Windows
   packaging process in [packaging/windows/package.ps1](../packaging/windows/package.ps1)
   and [releasing.md](releasing.md) when preparing a distributable build. A development
   branch push is not an application release.
3. Include both executables, required runtime files and licence notices in the
   bundle. Exclude local designs, recovery data, tokens, account settings and agent
   credentials. Record the artifact's SHA-256 checksum and source commit.
4. Verify extraction and launch on a clean staff account before widening the pilot.
   Provide agent-install instructions separately for testers using AI features.

No custom staff bundle or release has been produced by this documentation change.

## Acceptance checklist and feedback

Use a small disposable model for the first pass. For each PC, check:

- App launch, readable UI, viewport rendering, pan/orbit/zoom and window resizing.
- Sketch a constrained rectangle, extrude it, change a dimension and verify rebuild.
- Save a `.solidcraft` file, close and reopen it, and verify parameters and geometry.
- Export STEP/STL and reopen the export in SolidCraft where supported.
- SpaceMouse motion, stopping, connection changes and sketch navigation.
- Agent Connect/Test, read the design, create a simple block, change its dimension
  and undo the change. Confirm the CLI warning is absent on the new connection.

Capture feedback in a copy of this table; do not put staff credentials or private
designs in public issues.

| Field | Record |
|---|---|
| Build | Source commit, artifact name and SHA-256 |
| PC | Non-sensitive test identifier, Windows version, CPU/RAM |
| Graphics | GPU, graphics-driver version, display scaling |
| SpaceMouse | Model, USB/wireless, 3DxWare version |
| Agent | Adapter and version, connection result; no credentials |
| Reproduction | Steps, expected result, actual result |
| Evidence | Pass/fail, screenshot, relevant logs or crash report |

For a crash, use `solidcraft-cli crashes` and follow [crash-reports.md](crash-reports.md).
Review reports before sharing: they can include the design and reproduction data.
