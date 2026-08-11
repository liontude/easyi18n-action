# easyi18n sync — GitHub Action

Continuous i18n for projects whose translations live in one file (Apple
String Catalogs first). On every push that touches your catalog, this action
uploads it to [easyi18n](https://easyi18n.com), translates only what changed,
and opens a pull request with the translated catalog. Nothing to install,
nothing to run locally.

## Setup (once, ~2 minutes)

1. In the easyi18n dashboard: create a project (base language = your
   catalog's `sourceLanguage`), add target languages, enable the
   **String Catalog** output format, and create an API key with the
   `read`, `translate` and `publish` scopes.
2. Add the key as a repository secret named `EASYI18N_TOKEN`.
3. Commit an `easyi18n.yaml` at the repo root:

   ```yaml
   workspace: your-handle
   project: your-project
   format: xcstrings
   catalog: YourApp/Localizable.xcstrings
   ```

4. Commit this workflow as `.github/workflows/easyi18n.yml`:

   ```yaml
   name: easyi18n
   on:
     push:
       branches: [main]
       paths: ['**/*.xcstrings']
   concurrency: { group: easyi18n-sync, cancel-in-progress: false }
   permissions: { contents: write, pull-requests: write }
   jobs:
     sync:
       runs-on: ubuntu-latest
       steps:
         - uses: actions/checkout@v4
         - uses: liontude/easyi18n-action@v1
           with:
             token: ${{ secrets.EASYI18N_TOKEN }}
             max-credits: '50'
   ```

That's it. Write strings in Xcode, push, merge the PR that comes back.

## How it stays quiet

Re-syncing an unchanged catalog is a byte-exact no-op (the server publishes
nothing when the content digest matches), so the run triggered by merging the
action's own PR changes no files and opens no PR. Only real string changes
cost credits or create versions.

## Inputs

| Input | Default | Notes |
|---|---|---|
| `token` | — (required) | easyi18n API key (`eik_`) with `read`, `translate`, `publish`. |
| `config-path` | `easyi18n.yaml` | Where your config lives. |
| `max-credits` | unset | Abort if the estimated cost exceeds this many credits. |
| `cli-version` | pinned | `easyi18n_cli` version to install. |
| `github-token` | `github.token` | Token used to open the PR (see below). |

## If the PR isn't created

Repos where **Settings → Actions → General → "Allow GitHub Actions to create
and approve pull requests"** is off can't open PRs with the default token.
Either enable it, or pass a PAT with `repo` scope as `github-token`. The sync
itself still ran: the branch `easyi18n/sync` is pushed, and GitHub shows a
"Compare & pull request" banner.

## License

MIT
