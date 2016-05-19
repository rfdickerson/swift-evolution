# Package Manager Version Pinning

* Proposal: [SE-NNNN](https://github.com/ddunbar/swift-evolution/blob/master/proposals/NNNN-swiftpm-version-pinning.md)
* Author(s): [Daniel Dunbar](https://github.com/ddunbar)
* Status: **WIP**
* Review manager: **N/A**

## Introduction

This is a proposal for adding package manager features to "pin" or "lock"
package dependencies to particular versions.

## Motivation


## Proposed solution



## Detailed design

1. The build will always record the full set of version pin information. We will
   add a new command `export-pins` which will dump this information to the
   console or a file, for use in archival workflows, continuous integration, and
   debugging.

2. We will add a new command `pin ( [--all] | [<package-name>] [<version>] )`.

   This command pins one or all packages. The command which pins a single version
   can optionally take a specific version to pin to, if unspecified (or with
   `--all`) the behavior is to pin to the current package version in use.
   
   The pinning command itself creates a new data file adjacent to `Package.swift`
   called `PackagePins.json`, which records the package, the pinned version, and
   explicit information on the pinned version (e.g., the commit hash for the
   resolved tag).
   
3. We will add a new command `unpin ( [--all] | [<package-name>] )`.

   This is the counterpart to the `pin` command, and unpins one or all packages.

4. We will extend the workflow for `update` to honor version pinning.
   
   * This command errors if there are no unpinned packages which can be
     updated.
   
   * Otherwise, the behavior is to update all unpinned packages to the latest
     possible versions which can be resolved while respecting the existing pins.
   
   * Optionally, this command takes a `[--repin]` argument which can be used to
     lift the version pinning restrictions. In this case, the behavior is that all
     packages are updated, and packages which were previously pinned are then
     repinned to the latest resolved versions.
   
4. `Package.swift` will gain a new modifier on package dependencies, `usePins:
   true`.

   If enabled, then the package manager will inherit the pinned package
   information from the dependency when performing package dependency
   resolution. As a quality of implementation issue, we would like the package
   manager to report rich diagnostics when these additional requirements caused
   the graph to be unsolveable.

   The goal for allowing this flexibility in the manifest is so that developers
   can define their own workflow relationships with upstream dependencies. For
   example, a developer with an upstream dependency on a library which needs to
   explicitly micro-manage their own upstream dependencies may wish to simple
   inherit the "suggested" package versions from that library.

5. **TODO**: We have explicitly left out a workflow for manually specifying an
   extra set of pins to "overlay" on the build (conceptually the equivalent of
   `import-pins`). For now, users will always be able to manually pin to the
   versions represented in that file.

   The reason for leaving this out of the proposal is that we didn't want to
   undertake the design at this time of how the transient nature of needing to
   overlay a set of pins (for debugging a regression versus a historical set of
   build versions, for example) interacts with the pin version data which may be
   checked in alongside the project.

## Alternatives considered

**TBD**
