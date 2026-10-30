# `krew-index`

A personal [krew](https://krew.sigs.k8s.io/) index containing my own `kubectl` plugins.

[krew](https://krew.sigs.k8s.io/) is the plugin manager for `kubectl`. This repository acts as a
custom index that `krew` can be pointed at, so the plugins below can be installed with `kubectl krew
install`.

## Available plugins

| Plugin | Description | Source |
| ------ | ----------- | ------ |
| `tail` | Tail pod logs using [stern](https://github.com/stern/stern). | [davidcollom/kubectl-tail](https://github.com/davidcollom/kubectl-tail) |
| `watch` | Pass `kubectl` commands through [watch](https://man7.org/linux/man-pages/man1/watch.1.html). | [davidcollom/kubectl-watch](https://github.com/davidcollom/kubectl-watch) |

## Usage

Add this index to your krew configuration and install a plugin:

```bash
kubectl krew index add davidcollom https://github.com/davidcollom/krew-index
kubectl krew install davidcollom/tail
kubectl krew install davidcollom/watch
```

The `davidcollom/` prefix is required — plugins from a custom index have to be referenced with their
index name, so a bare `kubectl krew install tail` would be resolved against the default index instead.

This installs the `kubectl-tail` and `kubectl-watch` wrapper scripts, which `kubectl` exposes as the
`kubectl tail` and `kubectl watch` subcommands. Both are bash scripts that shell out to `stern` and
`watch` respectively, so those have to be installed separately.

To browse everything in the index:

```bash
kubectl krew search davidcollom/
```

## Layout

Each plugin is a YAML manifest under `plugins/`, named after the plugin:

```text
plugins/
├── tail.yaml    # kubectl-tail
└── watch.yaml   # kubectl-watch
```

The manifests follow the [krew plugin manifest format](https://krew.sigs.k8s.io/docs/developer-guide/plugin-manifest/),
with each one declaring the platform selector, the release `uri`, its `sha256` checksum, the binary
to install and the files to extract from the archive. `plugins/tail.yaml` in full:

```yaml
apiVersion: krew.googlecontainertools.github.com/v1alpha2
kind: Plugin
metadata:
  name: tail
spec:
  homepage: https://github.com/davidcollom/kubectl-tail
  shortDescription: Tail pod logs
  version: v0.0.1
  description: |
    Provides a simple way to easily tail pod logs using stern
  platforms:
    - selector:
        matchExpressions:
          - key: os
            operator: In
            values:
              - darwin
              - linux
      uri: https://github.com/davidcollom/kubectl-tail/releases/download/v0.0.1/release.zip
      sha256: 48b85b0e381fe414496d8a1a903f45a5e1cc7bad9200a51986ebaf4300867a39
      bin: kubectl-tail
      files:
        - from: kubectl-tail
          to: .
```

## Updating a plugin

When a plugin has a new release:

1. Update `version`, `uri` and `sha256` in the plugin manifest.
2. Compute the checksum of the release archive:

   ```bash
   shasum -a 256 release.zip
   ```

## License

No licence file is included in this repository.
