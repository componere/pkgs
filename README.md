# pkgs

This repository owns the reviewed policy and public keys for the static
native package repository at <https://pkgs.componere.dev>. Allowlisted producer
GitHub Releases are the authoritative package source. Publication runs
through one serialized writer in this repository.

Policy lives in [`.config/package-repository.yaml`](.config/package-repository.yaml).
Public keys live beside it under `.config/keys/`.

## Documentation

- [How to operate the package repository](docs/operations.md)
- [Release system reference](https://github.com/meigma/release/blob/0dee66ff6c4cc7e28d7bb65e97a37d701e0eff4a/docs/reference/release-system.md)
- [Operate a native package repository](https://github.com/meigma/release/blob/0dee66ff6c4cc7e28d7bb65e97a37d701e0eff4a/docs/how-to/operate-a-native-package-repository.md)
