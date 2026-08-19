# The Cogitorium plugin catalog

This is the index the plugin library inside Cogitorium reads. It holds one
small file listing where plugins live — never the plugins themselves.

## Listing yours

Open a pull request adding one entry to `plugins.json`:

```json
{
  "id": "release-radar",
  "name": "Release Radar",
  "author": "someone",
  "description": "Watches releases and files them into a workspace.",
  "repo": "someone/cogitorium-release-radar"
}
```

Five fields, and that is the whole schema. Everything else about a plugin —
what it needs, what it overrides, what it asks permission for — lives in its
own `plugin.yaml`, where it travels with the code instead of with a description
somebody wrote once and never updated.

**CI runs, and if it passes the entry merges on its own.** You are not waiting
for one of us to be awake.

## Publishing the code

The bundle stays in your repository, in your releases. Nothing here stores or
serves anybody's binaries, which is why this index costs one small file no
matter how many plugins are in it.

```sh
cogitorium plugins build .
# then tag a release and attach the zip it wrote
```

The file must be named `<id>.zip`, because that is the name every install
fetches:

```
https://github.com/<repo>/releases/latest/download/<id>.zip
```

Built by convention rather than by asking GitHub's API — the API needs a token
to be useful at any volume, and browsing a catalog should not depend on a
service being up to answer a question the URL already answers.

## What CI checks

| Check | Why |
|---|---|
| The diff touches only `plugins.json` and `verified.json` | A submission that also edits the workflows is not a submission |
| The file parses, with no field nobody implements | A field nobody reads is a field an author believes in |
| `id` is 3–48 lowercase characters, unreserved and unused | It becomes a template namespace and a URL prefix |
| `repo` is `owner/name` | It is what the download URL is built from |
| Every field is present | An entry with no description is a row nobody can read |
| The release exists and carries `<id>.zip` | An entry pointing at nothing is worse than no entry |
| The bundle installs and its templates compose | You learn here rather than from a stranger's log |

The last two are `cogitorium plugins check-bundle`, which is the same code the
server runs at boot — so a submission cannot pass CI and then fail to load.

## Editing or removing an entry

Those do not merge automatically, and this is the one rule worth explaining.

An edit can point an id that people have already installed at a **different
repository**, which hands that plugin's download URL to somebody else on every
install that has it. Nothing in a public JSON file can establish who owns an
id — the author of a pull request is whoever opened it, which is exactly the
claim under question. So an edit or a removal waits for a person.

Additions cannot do that, which is why additions are the thing that merges
itself.

## `verified.json`

A second list, of plugins somebody on the team has actually read:

```json
{ "id": "release-radar", "version": "1.2.0", "by": "eduard",
  "note": "reads a feed, writes nothing" }
```

**The mechanism is who may merge this file** — see `CODEOWNERS`. Anyone can add
themselves to `plugins.json`. Nobody can add themselves here.

A client shows three states rather than a badge: `verified` when the team read
the version you have, `verified-other-version` when they read a different one,
and `unchecked` when nobody has looked. The last is the ordinary state and not
an accusation. A missing or unreachable file leaves everything `unchecked`,
which is true rather than a guess in either direction.

**Verified means somebody read that version.** It is not a guarantee, not a
security audit, and not a substitute for approving the plugin on your own
install — which is where somebody who can actually see your data decides
whether this should touch it.

## `index.json`

Generated, never edited. A scheduled job asks each listed repository for its
current release and republishes the catalog with versions filled in, so a
client can tell an update exists by fetching one whole file — no query string,
no install id, no list of what it has. What your install runs stays your own
business by construction rather than by policy.
