# Scoop manifest fixes [![Tests](https://github.com/LogicDaemon/ScoopInstaller-manifest-fixes/actions/workflows/ci.yml/badge.svg)](https://github.com/LogicDaemon/ScoopInstaller-manifest-fixes/actions/workflows/ci.yml) [![Excavator](https://github.com/LogicDaemon/ScoopInstaller-manifest-fixes/actions/workflows/excavator.yml/badge.svg)](https://github.com/LogicDaemon/ScoopInstaller-manifest-fixes/actions/workflows/excavator.yml)

Fixed Manifests for [Scoop](https://scoop.sh), the Windows command-line installer. For manifests that are broken in some other bucket (and merging a PR there takes eternity).

## How do I install these manifests?

1. Add this bucket: run `scoop bucket add logicdaemon-fixes https://github.com/LogicDaemon/ScoopInstaller-manifest-fixes.git`.
2. To install, do `scoop install logicdaemon-fixes/<manifest>` (manifest name without `.json`, for example, `scoop install logicdaemon-fixes/ollama-full`).

## What's inside?

### Cursor

Fixes broken authentication links and separate configuration when opening `cursor://` protocol links.
The upstream manifest relies on passing `--user-data-dir` and `--extensions-dir` arguments through a shortcut. However, when the browser triggers the `cursor://` URL handler, Cursor doesn't know about these shortcut arguments and launches a new instance with default settings in `%USERPROFILE%\.cursor`, without any of the user settings. This fix removes CLI arguments, creating junctions for default paths instead, ensuring both direct execution and URL handlers use the correct Scoop-managed persistence directories.

### librehardwaremonitor

The upstream manifest lacks configuration persistence. This version adds pre-uninstall and post-install scripts to properly persist user configurations across updates and uninstalls. Additionally, it removes bulky debug symbols after installation.

### ollama-full

[`ollama-full`](https://github.com/ScoopInstaller/Extras/blob/master/bucket/ollama-full.json) and [`ollama`](https://github.com/ScoopInstaller/Main/blob/master/bucket/ollama.json) are basically the same, but `ollama` comes as a 2GB zip, while `ollama-full` is a 1GB InnoSetup executable. The extras bucket's `ollama-full` failed to extract for a few versions between 1.13 and 1.17.7 because of a too new InnoSetup used, incompatible with the extractor. This manifest works around that issue completely by running a silent installation to a staging directory, hardlinking the files, and uninstalling it. Currently the extras manifest works again, probably because the InnoSetup extractor was updated, but this manifest is still useful as this incompatibility will likely arise again.

### python

On my corporate notebook, there is a system-wide Python 3.10 installation in `%ProgramFiles%`, which is registered globally in the Windows Registry. This causes `py.exe` to only find that ancient system-wide install, unless the PEP 514 registry keys are added for the Scoop installation too. But in the main bucket manifest, it's a manual procedure both to add them and remove them before uninstalling.

My manifest imports the install reg file after installing, and the uninstall reg file before uninstalling.

### unison

Fixes an extraction path issue introduced in version 2.53.8 where the binaries are placed inside a nested subdirectory.

### chef-workstation

Fixes URL and regex for checkver, and works around execution failures that occur when running batch shims from different drives.
Introduces a custom shim implementation for batch files in the bin directory. This replaces the existing Ruby shims with a patched version that safely handles drive letter mismatches by pushing to the temporary directory when necessary.
There is also an embedded comment explaining how to use the knife without admin privileges.

### cinc-workstation

A free-as-in-beer, open source build of Chef Workstation. Includes the same batch shim fixes as `chef-workstation`, properly handling drive letter mismatches by pushing to a temporary directory when necessary.
