# Norca global registry

The default registry of [Rigger](https://github.com/bacanoo-io/norca/tree/main/rigger), the
package manager of Norca. In a stack, `source = "acme/redis"` is the module
`redis` of the namespace `acme` here, unless the stack names a registry
`acme` of its own.

This repository holds **entries, not code**: small generated JSON files
that say, for every listed version, where its source is (the repository,
tag and commit of its author), its tree digest and its signatures. The code
stays in the repositories of its authors. Rigger reads the files as they are;
there is no server.

```
modules/<first letter>/<namespace>/<name>.json   the versions of one module
stacks/<first letter>/<namespace>/<name>.json    the versions of one stack
namespaces/<namespace>.json                      the owner and the source repositories
keys/<namespace>/*.pub                           the signing keys of the namespace
advisories.json                                  warnings and blocked versions
```

An entry, shortened:

```json
{
  "namespace": "norca",
  "name": "nats",
  "versions": [
    {
      "version": "1.0.0",
      "repository": "github.com/bacanoo-io/norca-registry",
      "path": "modules/nats",
      "tag": "nats/v1.0.0",
      "commit": "…",
      "digest": "sha256:…",
      "signatures": [{ "key_id": "sha256:…", "signature": "MEUCIQ…" }],
      "listed": "2026-11-18"
    }
  ]
}
```

## What Rigger does with it

`rigger stack sync` reads the entry, takes the newest version that meets the
constraint and is not blocked, fetches it from the author's repository at
the listed commit, and refuses it unless its tree digest is the listed one
and a key of its namespace signs it. The lock file records the version,
the repository, the commit and the digest; a listed version never changes,
so a lock file stays valid for good. Warnings of `advisories.json` are shown
by `rigger stack sync` and `rigger stack plan`; a blocked version fails with its
reason.

A stack listed here is installed by its name (`rigger stack plan acme/data
-version "~> 1.2" -f my.vars.hcl`) or taken as a part of a local stack
(`stack "data" { source = "acme/data", version = "~> 1.2" }`); its entry
lists the modules its lock file pins, all of them listed here too.

`RIGGER_GLOBAL_REGISTRY` points Rigger to another registry of this shape: a git
address, or a directory such as a checkout of this repository.

## Getting listed

1. Ask for a namespace with the issue form *Request a namespace*: the
   owner, the repositories and their layout, and your public keys. A
   maintainer reviews it and adds `namespaces/<namespace>.json` and
   `keys/<namespace>/`.
2. Publish versions in your repository: `rigger module publish <dir> -key
   name.key` validates the module, signs its tree digest, commits the
   signature and tags the version (`<name>/v<version>`, or `v<version>`
   for a repository of one module); `rigger stack publish <dir> -key
   name.key` does the same for a stack with `metadata.hcl` and its lock
   file. Push the tag.
3. The *Update* workflow lists every new tag of your repositories once it
   passes the checks: the module validates, its metadata names the tag's
   name and version, a key of your namespace signs it, it is not blocked.
   A tag that fails stays out and is shown in the summary of the run.

## Checks

`rigger index check` runs on every pull request and push: every file in its
format, every key parses, every entry belongs to a namespace, and the
change only adds: a listed version or an advisory is never changed or
removed (POLICY.md). Every version a change adds is checked against its
source. Once a week every listed version is.

The same commands run locally on a checkout:

```shell
rigger index update -dir . [-namespace acme]
rigger index check -dir . [-base origin/main]
```

`-replace url=path` reads a repository from a local clone (its entries still
name the URL).

## The namespace norca

`norca` holds the official modules, from
[norca-registry](https://github.com/bacanoo-io/norca-registry). Its
signing key is made by the maintainers (`rigger key generate -out norca`): the
public half is `keys/norca/norca.pub`, the private half is never in a
repository. A version of `norca` is listed once that key signs it.

## License

The files of this repository are under the Mozilla Public License 2.0
(`LICENSE`). Every listed module and stack has the license of its own
repository.
