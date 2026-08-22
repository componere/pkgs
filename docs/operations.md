# How to operate the package repository

Replay a verified producer GitHub Release into the public repository at
<https://pkgs.componere.dev>, inspect the result, and recover a failed run.

## Prerequisites

- permission to run [`.github/workflows/publish.yml`](../.github/workflows/publish.yml)
  in this repository
- a published GitHub Release for an allowlisted producer and exact stable tag
- the reviewed policy in
  [`.config/package-repository.yaml`](../.config/package-repository.yaml)

Producers never receive R2 credentials or aggregate signing keys. Those
secrets stay in this repository's `packages-production` environment. A
producer sends only `{repository, tag}`.

## Replay a publication

Run **Publish package release** with the producer repository and exact tag.

From the Actions tab, enter:

- `repository`: GitHub `owner/name`, such as `componere/incusos-builder`
- `tag`: the published stable tag, such as `v0.2.0`

From the CLI:

```bash
gh workflow run publish.yml \
  --repo componere/pkgs \
  -f repository=componere/incusos-builder \
  -f tag=v0.2.0
```

Replace the repository and tag with the published producer release you are
replaying. The workflow calls
`meigma/release/.github/workflows/publish-package-repository.yml` at
`0dee66ff6c4cc7e28d7bb65e97a37d701e0eff4a` (v0.1.17) and selects
`packages-production`. Concurrent production writes share the
`package-repository-production` group and are not cancelled.

A producer `repository_dispatch` uses the same path: event type
`package-release` with exactly `{repository, tag}`.

## Read the result

A successful CLI publication writes `state` in its JSON result:

- `published` when at least one object was uploaded
- `unchanged` when every generated object already matched R2

The publish step log contains that envelope. Both states are success.

## Check the public roots

After a successful run, fetch the three commit roots:

```bash
curl --fail --silent --show-error \
  https://pkgs.componere.dev/apt/dists/stable/InRelease >/dev/null
curl --fail --silent --show-error \
  https://pkgs.componere.dev/rpm/stable/x86_64/repodata/repomd.xml >/dev/null
curl --fail --silent --show-error \
  https://pkgs.componere.dev/apk/stable/main/x86_64/APKINDEX.tar.gz >/dev/null
```

A reachable root means that format's metadata was committed; it is not a
client installation check.

## Recover a failed run

Do not delete or rename objects in R2. Fix the failed prerequisite, then
replay the same `{repository, tag}` pair.

Publication is convergent:

- matching immutable objects are skipped
- missing objects are uploaded
- replaceable metadata is regenerated and uploaded
- a different value at an immutable path fails instead of overwriting that
  object

A successful replay reports `state: unchanged` when every generated object
already matches R2.

## Rotate a key

Published public-key objects are immutable. A rotation adds a new reviewed
`published` filename in `.config/package-repository.yaml`, commits the new
public key under `.config/keys/`, and updates the corresponding private key in
the `packages-production` environment (aggregate keys) or the producer's Actions
secrets (producer keys). Never reuse a published key filename for new key
material.
