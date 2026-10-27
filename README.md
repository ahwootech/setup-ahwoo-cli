# setup-ahwoo-cli

Installs the [Ahwoo](https://ahwoo.com) CLI and adds `ahwoo` to the `PATH`, so a workflow
can push game builds to Ahwoo. Runs on Linux x64, macOS arm64 and Windows x64 runners.

## Usage

```yaml
- uses: ahwootech/setup-ahwoo-cli@v1
```

| Input | Description |
|-------|-------------|
| `version` | CLI version to install, such as `2026.10.12.625`. Defaults to the latest release. |

The action downloads the CLI from `https://install.ahwoo.com/cli/` and checks it against
the release's SHA-256 checksums before installing it.

## Push a build from CI

Ask Ahwoo for a client ID and secret and your package ID. Store the client ID and secret
as repository secrets. Then, in the job that builds your game, add these steps after the
build:

```yaml
jobs:
  build:
    runs-on: ubuntu-latest
    steps:
      # Your checkout, build and zip steps.

      - uses: ahwootech/setup-ahwoo-cli@v1

      - name: Push to Ahwoo
        env:
          AHWOO_CLIENT_ID: ${{ secrets.AHWOO_CLIENT_ID }}
          AHWOO_CLIENT_SECRET: ${{ secrets.AHWOO_CLIENT_SECRET }}
        run: >
          ahwoo push --ci
          --package <PACKAGE_ID>
          --version <VERSION>
          --target windows_x64=./build.zip
```

`--version` is the name players see for this build. Use any name you like, such as
`1.4.0` or `Cicada`, but don't reuse one: `push` rejects a version the package already
has.

Supply one `--target` for each build target your package has. `push` waits until the
build finishes processing and exits non-zero if it fails. Add `--detach` to return once
the upload is confirmed. Run `ahwoo push --help` for every option.

### Push several packages

Each `ahwoo push` creates one build for one package. To publish several packages, such
as a game and its dedicated server, run `ahwoo push` once for each, with that package's
ID and archives:

```yaml
      - name: Push to Ahwoo
        env:
          AHWOO_CLIENT_ID: ${{ secrets.AHWOO_CLIENT_ID }}
          AHWOO_CLIENT_SECRET: ${{ secrets.AHWOO_CLIENT_SECRET }}
        run: |
          ahwoo push --ci --package <GAME_PACKAGE_ID> --version <VERSION> --target windows_x64=./game.zip
          ahwoo push --ci --package <SERVER_PACKAGE_ID> --version <VERSION> --target linux_x64=./server.zip
```

## Other CI systems

Install the CLI with the same scripts the action uses:

```bash
curl -fsSL https://install.ahwoo.com/cli/install.sh | sh
```

```pwsh
irm https://install.ahwoo.com/cli/install.ps1 | iex
```

Set `AHWOO_CLI_VERSION` to pin a version and `AHWOO_CLI_INSTALL_DIR` to choose where
`ahwoo` goes.
