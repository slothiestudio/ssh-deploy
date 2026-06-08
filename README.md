# ssh-deploy

A composite GitHub Action that deploys a built site to a server over **SSH using
rsync**. A self-hosted replacement for third-party FTP/SSH deploy actions.

- SSH **key** or **password** auth (password via `sshpass`)
- Encrypted keys supported via `passphrase` (loaded through `ssh-agent`)
- **Host-key fingerprint pinning** (MITM-safe), with trust-on-first-use fallback
- `rsync --archive --delete` with configurable, comment-friendly exclude lists
- Excluded paths are **protected from `--delete`** — live data on the server is never removed
- Optional `post-deploy` command run on the server (e.g. clear a cache)

## Usage

```yaml
- name: Deploy
  uses: slothiestudio/ssh-deploy@v1
  with:
    host: ${{ secrets.SERVER_URL }}
    username: ${{ secrets.USERNAME }}
    target: /var/www/example.com
    ssh-private-key: ${{ secrets.SSH_KEY }}
    fingerprint: ${{ secrets.SSH_FINGERPRINT }} # recommended
    extra-exclude: |
      /content
      /storage
```

### Password auth instead of a key

```yaml
  with:
    host: ${{ secrets.SERVER_URL }}
    username: ${{ secrets.USERNAME }}
    target: /var/www/example.com
    password: ${{ secrets.PASSWORD }}
```

### With a post-deploy step

```yaml
  with:
    # ...auth + target...
    post-deploy: rm -rf /var/www/example.com/storage/cache/*
```

## Inputs

| Input | Required | Default | Description |
|-------|----------|---------|-------------|
| `host` | yes | | SSH host / server address. |
| `username` | yes | | SSH username. |
| `target` | yes | | Absolute path to the deploy directory on the server. |
| `port` | no | `22` | SSH port. |
| `ssh-private-key` | no | | Private SSH key (PEM). Preferred auth. |
| `passphrase` | no | | Passphrase for an encrypted private key. |
| `password` | no | | SSH password. Used only when no key is given (via `sshpass`). |
| `fingerprint` | no | | Expected host-key fingerprint (`SHA256:...`). When set, verified before connecting; when empty, TOFU. |
| `timeout` | no | `30` | SSH connection timeout (seconds). |
| `source` | no | `./` | Local directory to deploy (trailing slash = "contents of"). |
| `delete` | no | `true` | Pass `--delete` so files removed locally are removed on the server. |
| `exclude` | no | _(see below)_ | Generic exclude patterns (VCS, editor, OS noise, deps). |
| `extra-exclude` | no | | Project-specific exclude patterns, merged with `exclude`. |
| `post-deploy` | no | | Command run on the server after the sync (with `set -e`). |

Auth precedence: if `ssh-private-key` is set it is used (plus `passphrase` if the key is
encrypted); otherwise `password` is used; otherwise the job fails.

### Exclude format

Newline-separated patterns. Lines starting with `#` are comments. A leading `/` anchors
the pattern to the deploy root. Excluded paths are skipped on upload **and** protected
from `--delete`, so live data (content, caches, sessions, licenses) is never wiped.

Default `exclude`:

```
/.git
/.github
.git*
node_modules
.idea
.vscode
.claude
.DS_Store
Icon
```

Keep these generic; put project-specific paths in `extra-exclude`.

## Getting the host fingerprint

```sh
ssh-keyscan -p 22 your.host | ssh-keygen -lf - | awk '{print $2}'
```

Store the resulting `SHA256:...` value as the `fingerprint` secret.

## Using from a private action repo

This action is **private**. For other `slothiestudio` repos to consume it, enable it in
this repo's **Settings → Actions → General → Access →
"Accessible from repositories in the slothiestudio organization"**.

## Versioning

Reference a major tag (`@v1`) for automatic minor/patch updates, or pin an exact tag
(`@v1.0.0`). The `v1` tag is moved forward on each backwards-compatible release.

## License

MIT © slothiestudio
