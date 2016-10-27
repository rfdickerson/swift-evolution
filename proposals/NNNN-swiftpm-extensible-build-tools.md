# Package Manager External Targets

* Proposal: [SE-NNNN](https://github.com/ddunbar/swift-evolution/blob/master/proposals/NNNN-swiftpm-externaltargets.md)
* Author(s): [Daniel Dunbar](https://github.com/ddunbar)
* Status: **WIP**
* Review manager: **N/A**

**This is a WORK IN PROGRESS.** *It is currently just intended to be used for discussion.*

## Introduction

This is a proposal for adding package manager support for building *external
targets*, i.e., targets for which the build process is not able to be understood
by the Swift package manager directly.

## Motivation

There are large bodies of existing, complex, software projects which cannot be
described directly in SwiftPM, but which other projects wish to depend upon.

The package manager currently supports two mechanisms by which non-SwiftPM
projects can be used:

* The system module map feature allows defining support for a package which
  already exists in an installed form on the system. In conjunction with system
  package managers, this can be used to configure an environment in which a
  package can depend on such a body of software.

* The C language targets feature (itself a work-in-progress currentyl) allows
  the package manager to build and include C family targets. This can provide
  similar features to the previous bullet, but through use of the standard C
  compiler features (like header and library search paths). The external
  project again needs to be installed using a different package manager.

These mechanisms have two major downsides:

1. The usability of a package depending upon them requires interaction with a
   system package manager. This increases developer friction.

2. The system packages are global. This means project builds are no longer self
   contained. That introduces greater complexity when managing the dependencies
   of multiple projects which need to run in the same environment, but may have
   conflicting dependencies. This is compounded by the package manager itself
   not understanding much about the system dependencies.

## Proposed solution



## Detailed design

**TBD**

## Alternatives considered

**TBD**
