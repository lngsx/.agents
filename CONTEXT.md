# dotagents

A personal repo of agent tooling and conventions: knowledge files that agent
files and skills are composed from, plus scripts that build and serve
llms.txt files for libraries.

## Language

**Knowledge file**:
A stored copy of conventions or reference material in `knowledge/`, kept
exactly as it arrived (pasted or hand-written). Never edited in place.
_Avoid_: doc, source of truth (that role belongs to the whole file)

**Index**:
`knowledge/index.md` — the single listing of everything in `knowledge/`.
If it's not on the index, it doesn't exist.
_Avoid_: table of contents, manifest

**Caveat**:
An entry in `knowledge/caveats.md`: a context warning about a knowledge file
whose copied content doesn't hold in every project it may be composed into.
Copies stay untouched; warnings live here.

**Compose**:
Creating a new agent file by copying sections from knowledge files exactly as
written, optionally filtered by the user's scope.

**Blend**:
Appending only the missing conventions to an existing agent file the user
provides. Never rewrites, reorders, or resolves conflicts — lists them.
_Avoid_: merge, combine, magically blend

**Staging**:
`staging/` at the repo root, gitignored — where composed output lands for the
user to review and hand-copy to its real destination.
_Avoid_: output, dist

**Copy-paste noise**:
Site artifacts and HTML escapes that arrive with pasted page content. Pointed
out, never removed without an explicit ask.
