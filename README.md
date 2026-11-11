# Curse mods

The registry of what is made for [Curse](https://curse-rusts.github.io/): mods (`.hmod`), native mods (`.cmod`)
and resource packs, each one reviewed before it is listed. The entries live here; the files they list are release
assets of this repository.

The registry is not deployed yet. The game and the launcher have no index address, so nothing is fetched from here
at runtime and a fresh install plays like vanilla. Mods installed by hand work today: `curse-mods/` for content
mods, `curse-native-mods/` for native bundles. Browsing and in-game install arrive when the registry has an address,
not before.

**What a file is.** A content mod is data and assets only, with no code, and runs in the game's mod sandbox with no
file, network or clock access beyond what its page lists. A native mod is Rust that changes the game's own source
before it is compiled; it runs with the player's account permissions (see the
[mods page](https://curse-rusts.github.io/mods.html)). Its bundle pins the exact Curse and mixin-engine revisions
used for the build; other revisions have not been validated, and every version of a native mod is reviewed, owner
or not.

**Who is responsible.** Except the projects marked Official (made by the Curse project), everything here is made
and owned by its own authors, not by the Curse project or its maintainers. A listing is not an endorsement. See the
[mod policy](MOD-POLICY.md): what authors promise when they submit (ports of other mods included), the licences it
accepts, and how to have a project taken down.

## Submitting

A submission is a pull request that adds `mods/<id>.toml` and puts the file in the `staging` prerelease as
`<sha256>.<extension>`. Then:

1. This repository's checks run on it: the file's own checks, a trial run of a mod in the sandbox, and the index it
   would build. Their report is in the pull request's checks.
2. A reviewer (see [`.github/CODEOWNERS`](.github/CODEOWNERS)) reads the entry, the source and the report, and asks
   for changes in the pull request if needed.
3. Merging publishes it: the file becomes a release, and the index gains the project.

What the registry takes now: mods, modpacks of registry mods, and resource packs. The checks are the registry
commands of `cargo curse-mod`, the tool that ships with the game's source:

```sh
cargo curse-mod check target/curse-mod/my_mod.hmod
cargo curse-mod registry build . --files files --out index.json
```

**Updates** come the same way: the pull request adds a version to the entry, and a reviewer reads it before it is
published. Only an entry's `owners` can add versions. A published file never changes: publish a new version.

**Signing** is optional and recommended: put your [minisign](https://jedisct1.github.io/minisign/) public key in
your entry, and every later version must be signed with it, so nobody else can publish an update in your name.

## Reporting

Something wrong with a project? [Report it](../../issues/new?template=report.yml). Your work published without
your permission? [Ask for a takedown](../../issues/new?template=takedown.yml) ([mod policy](MOD-POLICY.md)). A
project that does harm is taken down in [`revoked.toml`](revoked.toml): the launcher then refuses it and turns
installed copies off, telling players why.

## Layout

| Path | What |
| --- | --- |
| `mods/<id>.toml` | One project: its page and its versions, each pinned by SHA-256 |
| `icons/<id>.png` | Its icon, at most 256 KiB |
| `revoked.toml` | Files taken down |
| `.github/workflows/check.yml` | A pull request's checks |
| `.github/workflows/publish.yml` | On merge: releases, then `index.json` |
| `.github/workflows/cleanup.yml` | Weekly: staging files nobody needs any more |
| `.github/workflows/native-compat.yml` | Native mods on the newest Nightly, as the launcher builds them: a pull request's new version applied and compiled with the most downloaded native mods, its module names against every mod's; daily, every mod's newest version, the ones that don't apply marked `broken_on` in the index (`scripts/native_compat.py`) |
| `.github/workflows/native.yml` | Curse's native mods: each changed one's tests, `.hnmod` and game check, all of them together, and weekly on the newest Nightly |
| `native/<name>/` | Curse's own native mods (Zoom, FPS Counter, Keystrokes, Minimap, Recipes, Coordinates HUD): the source of their bundles |
| `native/tools/` | Shared by them: apply on the game, check, build, package, prove (`native/tools/native.py`) |
| `scripts/` | What the workflows share |

The format of the entries and of `index.json` is the game's own `docs/api/mod-index.md`. Entries are read by the
registry commands of `cargo curse-mod` at the revision in the repository variable `HORIZON_REV` (else `main`); the
variable keeps its name because the tool it names has not moved.

## Curse's native mods

`native/<name>/` holds the source of Curse's official mods, as [curse-mixins](https://github.com/curse-rusts/curse-mixins)
bundles on the game's native hook points. Each folder keeps the history of the repository it came from. A mod
builds on any game whose hook points are at least the version its `compatibility.json` names; `horizon_revision` is
the commit it was last tested on. `smp` and `voicechat-server` are server mods, built into the official server's
engine (see their READMEs).

```sh
python3 native/tools/native.py check zoom          # apply on the tested revision and cargo check the game
python3 native/tools/native.py check all           # every native mod together
python3 native/tools/native.py package all         # native/target/<id>-<version>.hnmod, for a registry entry
```

`proof` drives the built game; each mod's README has its command.

The bundles in `native/` still name their manifests `horizon-mixin.toml` and their packages `horizon-native.json`:
the game reads those names, and older launchers read nothing else. Newer tools also write `curse-mixin.toml` and
`curse-native.json`, which the registry accepts.

To make your own, start from
[curse-native-mod-template](https://github.com/curse-rusts/curse-native-mod-template).
