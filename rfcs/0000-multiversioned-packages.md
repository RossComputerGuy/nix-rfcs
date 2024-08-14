---
feature: Multiversion Packages
start-date: 2024-08-13
author: Tristan Ross (@RossComputerGuy)
co-authors: Morgan Jones (@numinit), Dan Baker (@djacu)
shepherd-team: (names, to be nominated and accepted by RFC steering committee)
shepherd-leader: (name to be appointed by RFC steering committee)
related-issues: https://github.com/NixOS/nixpkgs/pull/331011, https://github.com/NixOS/nixpkgs/pull/325175, https://github.com/NixOS/nixpkgs/pull/334541
---

# Summary
[summary]: #summary

Nixpkgs contains many packages which often have multiple versions of those packages. Nixpkgs should have a standard way of defining packages which offer multiple versions.

# Motivation
[motivation]: #motivation

As nixpkgs grows, a well defined and common structure to defining packages makes it less burdensome to maintain.

# Detailed design
[design]: #detailed-design

Create a `default.nix` in nixpkgs in the location for the package. Copy this template:

```nix
{
  mkVersions,
  extraVersions ? {},
}:
mkVersions {
  versions = {} // extraVersions;
  package = ./package.nix;
}
```

Then change the name of `extraVersions` and `extraPackages`, replace `extra` with the name of the package. This can then be imported in `pkgs/top-level/all-packages.nix` using `callPackages`. Versions should be defined as an attribute set which can be interpreted like a map. The key will be the full version name which will be fetched from source. The value will be an attribute set which changes functionality and provides the source hash.

The package's `package.nix` will then follow the typical design of a nix package definition. However, it will use `lib`'s version checking functions to enable, disable, or change functionality based on the version. It is heavily recommended to use the `finalAttrs` style with `stdenv.mkDerivation` instead of `rec`.

It is acceptable to place the attribute set of versions inside a specific `version.nix` or `version.json` file for automatic updates. However, the additional versions should be combined in order to produce the base and additional versions attribute set.

# Examples and Interactions
[examples-and-interactions]: #examples-and-interactions

A precursor design was developed for Flutter, this evolved into the design used in LLVM and Zig as implemented by Tristan Ross (@RossComputerGuy). LLVM has benefitted greatly as the number of files having duplicate changes have gone down.

# Drawbacks
[drawbacks]: #drawbacks

This design is rigid and not always flexible, it also may require many version checking calls which increases complexity and reduces readability.

# Alternatives
[alternatives]: #alternatives

The alternative is to have a generic package nix file and a nix file for each version. Each version file then uses `callPackage` on the generic package nix file. However, this increases the number of files.

# Prior art
[prior-art]: #prior-art

Discourse has had no discussions.

# Unresolved questions
[unresolved]: #unresolved-questions

Handling automatic updates with @r-ryantm.

# Future work
[future]: #future-work

Packages such as Android tools, GCC, rustc, and various other compilers could benefit with this.
