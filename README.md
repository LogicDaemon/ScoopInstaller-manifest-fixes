# Scoop manifest fixes [![Tests](https://github.com/LogicDaemon/ScoopInstaller-manifest-fixes/actions/workflows/ci.yml/badge.svg)](https://github.com/LogicDaemon/ScoopInstaller-manifest-fixes/actions/workflows/ci.yml) [![Excavator](https://github.com/LogicDaemon/ScoopInstaller-manifest-fixes/actions/workflows/excavator.yml/badge.svg)](https://github.com/LogicDaemon/ScoopInstaller-manifest-fixes/actions/workflows/excavator.yml)

Fixed Manifests for [Scoop](https://scoop.sh), the Windows command-line installer. For manifests that are broken in some other bucket (and merging a PR there takes eternity).

How do I install these manifests?
---------------------------------

To add this bucket, run `scoop bucket add scoopfixes https://github.com/LogicDaemon/ScoopInstaller-manifest-fixes.git`. To install, do `scoop install scoopfixes/<manifest>`.
