# Package Manager Multi-Package Repositories

* Proposal: [SE-NNNN](https://github.com/apple/swift-evolution/blob/master/proposals/NNNN-swiftpm-multi-package-repos.md)
* Author(s): [Daniel Dunbar](https://github.com/ddunbar)
* Status: **Concept**
* Review manager: TBD

## Introduction

The package manager currently expects each repository can contain a single
package. This is a proposal for adding package manager support for repositories
which contain multiple packages.

## Motivation

Large teams and organizations often have many projects coexisting in one
repository. In some organizations, this can even take the form of *all* projects
being in a single repository (a "monorepo"). For other organizations, for
example LLVM + Clang + Swift itself, this may take a much more limited scope of
related projects split among repositories in an ad hoc manner.

Adoption of the Swift package manager in these environents requires some ability
for packages to (a) not be tied to being in the root of the repository, and (b)
be able to co-exist in the same repository, without forcing everything to be in
one package.

## Proposed solution

Our proposed solution is to allow an arbitrary organization of packages within a
repository, which we will identify as packages by the reachability of their
`Package.swift` from the repository root. We will refer to packages not at the
root as _subpackages_ of the repository.

Each subpackage in the repository will be required to have a unique name defined
by the package name in the manifest. This will be the name used to refer to the
subpackage in other dependency specifications.

We will treat each subpackage as having a version matching the versions derived
from the tag information in the repository itself; thus all subpackages in a
multi-package repository have the same versions.

When performing dependency resolution on a package graph, we will unify all of
the requirements on any individual repository across all requested subpackages,
to ensure only a single revision of the multi-package repository is required.

## Detailed design

1. A multi-package repository is defined as one which *does not* contain a
   `Package.swift` at the root, and which *does* contains multiple
   `Package.swift` files accessible via a path in the repository that *does not*
   pass through any directory defined by the package manager's convention-based
   layout to contain code. That is, the following is a multi-package repository
   with two subpackages:

   ```
   ROOT/Foo/Package.swift
   ROOT/Bar/Package.swift
   ```
       
   while this:

   ```
   ROOT/Foo/Package.swift
   ROOT/Foo/Sources/Foo/Package.swift
   ```

   is a multi-package repository with one package.

   This definition applies *without regard* to the contents of any
   `Package.swift` manifest, it can be computed strictly from the file system
   organization within the repository. This implies that one cannot use the
   `exclude` mechanism to hide subpackages. We do not view that as a significant
   problem, since the conventions typically would not have matched any other
   directory.

2. Any package *not* located at the root of the repository is referred to as a
   *subpackage* of that repository.

3. We will add support to the manifest dependency declaration for a `subpackage`
   specifier identify the `name` of the subpackage of a multi-package repository
   the dependency is on:

   ```
   let package = Package(
       dependencies: [
           .Package(url: "https://example.com/project", subpackage: "Foo", version: ...)
       ])
   ```

   The name here is the name assigned to the subpackage within the resolved
   version of the repository.

4. We will add support to the manifest dependency declaration for *only*
   specifying a subpackage:

   ```
   let package = Package(
       dependencies: [
           .Package(subpackage: "Foo")
       ])
   ```

   In this case, the package will be assumed to be within the same
   repository. There is no version specifier in this form, as the version is
   inherently locked to that of the referencing dependee subpackage.

5. We will amend the definition for existing package dependencies to require the
   URL they identify to be a mono-package repository (when no subpackage
   specifier is present). We will allow a dependency to refer to a mono-package
   repository with a subpackage specifier matching the name of the sole package,
   to assist with repositories wishing to transition to (or from) a
   multi-package repository.

6. We will add support for efficiently scanning a repository to identify all
   subpackages. For dependencies, we can implement this purely by operating on
   the underlying Git object store without needing to touch the file system,
   which should make this efficient even for large repositories. For very large
   repositories or other SCM systems, we will most likely eventually need to
   take care to cache this information.

7. For resolving subpackage names, we will need to parse the manifest for each
   subpackage. This may be a very time consuming operation, so we will reserve
   the right to heuristically determine the "likely" subpackage and parse its
   manifest first. If that manifest matches, we will avoid loading any other
   subpackage manifests we do not need. In practice, this should suffice to be
   efficient in almost all cases, but it does mean we will *not* diagnose
   non-unique subpackage names on dependent repositories. This is viewed as an
   acceptable tradeoff for the performance benefit in resolving subpackages.

8. When resolving local subpackage dependency references (references with no
   URL), we will need to be able to identify the root of the repository (in
   order to then be able to enumerate the other subpackages to resolve the
   name). Previously the package manager would do this by simply looking for
   `Package.swift`, but we will need to rely on an ability to infer this based
   on the SCM system itself.

9. The behavior of local package operations, like `swift build`, will continue
   to operate on a single package at a time. We will evaluate building
   additional caching for repository-level data (e.g., the cache of known
   subpackages) as the need presents itself, but will still create separate
   build arenas for each individual package.


## Impact on existing packages

None, this was not previously possible.


## Alternatives considered

We believe we must support this feature in one form or another to support widespread adoption of the package manager.

We considered alternatives in several areas:

1. We considered whether the appropriate mechanism for specifying a subpackage
   should be the path inside the repository, or by a canonical name. The major
   benefit of using a path is that it allows resolving the subpackage without
   needing to search the repository for packages. However, it has a significant
   downside in that clients will be broken by repository reorganization (which
   would effectively count as a major semantic version change).

2. We debated how version data should be associated with subpackages. From a
   strict semver perspective, it is incorrect to associate one semver with the
   entire reposity, as packages which are not changing should technically stay
   at their last semver (following the rule that a semver should *ONLY* change
   in response to the rules, for maximal compatibility). However, there is no
   clear alternative to this behavior, as we only have a limited set of ways we
   can extract versioning information from a bare repository without resorting
   to complicated tricks (like embedding sideband metadata inside the
   repository).

   In addition, it would be fairly unexpected (and potentially non-performant)
   to need to checkout multiple versions of a large repository simply to satisfy
   a need for different versions of its subpackages.

3. We considered introducing an explicit marker for the root of the workspace
   (e.g., `Packages.swift`, which could also serve as a place to store
   additional metadata on the packages). However, we did not see sufficient need
   to warrant its introduction.
