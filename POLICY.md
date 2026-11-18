# Policy

The rules of the registry, as the design of Rig sets them (section 7.3)
after the policy of the OpenTofu registry.

## Listed versions are never changed

- A version is listed only after its checks passed: nothing is listed
  unchecked.
- A listed version is never updated and never deleted: its source, commit,
  digest and signatures stay as they were accepted. A lock file that names
  it stays valid for good, and what was reviewed is what is installed.
- The only change is a new version, which goes through the same checks. A
  faulty version is answered by a newer one.
- The checks of every pull request and push refuse a change or a removal of
  a listed version, and of an advisory.

## Removal, only in four cases

A version is taken out only for:

1. malicious code or a supply chain compromise,
2. exposed secrets,
3. a compromised signing key,
4. a legal requirement.

It needs a request with evidence (the issue form *Report a version*) and a
recorded decision of the maintainers. The version is not deleted from its
entry: it goes onto `advisories.json` with the level `blocked`, its reason
and the link to the decision. It is not offered again, and a lock file that
names it fails with that reason. Never for a mistake of CI, a small bug or a
change of mind.

## Advisories

`advisories.json` lists warnings and blocks: kind, namespace, name, version,
level (`warning` or `blocked`), reason, and for a block the decision. A
warning is shown by `rig stack sync` and `plan` and changes nothing else.
Advisories are added, never taken back.

## Namespaces and keys

A namespace belongs to the user or organization that owns its source
repositories. Namespaces, their keys and their repositories are added by
the maintainers of the registry after the review of a request, not by
direct pull requests. A compromised key is removed from `keys/`, and the
versions it signed are reviewed under the third case above.

## Stacks

A stack listed here builds on the registry: every registry module it uses
must itself be listed (any namespace), at the version its lock pins. A
stack that reaches for a module outside the registry is not accepted.
