# Task description

Rewrite `bucket/ollama-full.json` so Scoop no longer relies on `innosetup: true` extraction for `OllamaSetup.exe`. The manifest should run the installer silently into a subdirectory of `$dir`, hardlink files into `$dir` (falling back to copying when hardlink creation fails), then run the copied uninstaller from `%TEMP%` so the normal uninstall registry entries are removed while the Scoop-managed files remain available. Keep `checkver` and `autoupdate` behavior intact.

# Current state analysis

- The current manifest is copied from `extras` and uses `innosetup: true` plus architecture-specific cleanup of extracted `,1` and `,2` files.
- `OllamaSetup.exe` does not unpack correctly through Scoop's Inno extraction path, so installation is currently broken.
- Inno Setup docs confirm the needed silent install switches (`/SP-`, `/VERYSILENT`, `/SUPPRESSMSGBOXES`, `/NORESTART`, `/DIR=`) and silent uninstall switches (`/VERYSILENT`, `/SUPPRESSMSGBOXES`, `/NORESTART`).
- The Scoop app repository is now available in the workspace, so `current/schema.json` can be consulted as the field reference if needed.
- The temporary installer staging directory should use a descriptive name (`tempinstall`) instead of a generic `app` subdirectory.
- The staged installation directory must not be deleted manually after the uninstall attempt. If the Inno uninstaller fails, that directory is needed to complete cleanup later and remove the leftover uninstall registry records.
- Running the uninstaller "from %TEMP%" is satisfied by using `%TEMP%` as the working directory; copying `unins*` into a temporary folder is unnecessary for this manifest.
- The upstream `ollama.iss` confirms that the model-removal checkbox is only shown when `not UninstallSilent`. Running the uninstaller with `/VERYSILENT` keeps `DeleteModelsChecked` false, so models are preserved.
- The upstream `[Run]` entry launches `ollama app.exe` with `Flags: postinstall nowait runhidden` and does not include `skipifsilent`, so there is no documented command-line switch to disable the autostart during silent install.
- Using PowerShell `Start-Process -Wait` against the setup EXE can effectively wait on the launched child app processes too, which blocks the Scoop install flow. The manifest should wait only for the setup process itself, then terminate the spawned Ollama processes before running the uninstaller.

# Step-by-step plan

1. [done] Inspect the broken manifest, relevant instructions, and working manifest examples.
2. [done] Design a replacement install flow that avoids `innosetup: true` and keeps Scoop ownership in `$dir`.
3. [done] Patch `bucket/ollama-full.json` with a custom installer/uninstaller flow while preserving `checkver`/`autoupdate`, using `tempinstall` as the staging subdirectory and preserving it when uninstall cleanup fails.
4. [done] Validate the edited manifest structure and schema compatibility using the available local schema and editor diagnostics.
5. [done] Append the validation result summary to this file.

# Summary

- Replaced the broken `innosetup: true` extraction approach with a custom `installer.script` flow.
- Installs Ollama silently into `$dir\tempinstall`, mirrors the installed files into `$dir` by hardlink where possible and copy as fallback, and runs the original Inno uninstaller with `%TEMP%` as its working directory.
- Avoids hanging on the auto-launched Ollama processes by waiting for the installer process with `PassThru`/`WaitForExit()`, then killing `ollama app.exe` and `ollama.exe` before the silent uninstall step.
- The staged `tempinstall` directory is intentionally preserved if the uninstall step returns a non-zero exit code, so cleanup can be completed later and uninstall registry records do not become orphaned permanently.
- `checkver` and `autoupdate` kept intact.
