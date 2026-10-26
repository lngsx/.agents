# AGENTS.md

`.agents` is dotfiles for agents — the portable tooling and config I keep for my
own machine and can move or share anywhere. It's a home for small, focused tools
that turn some source into something an LLM can use. Each task gets a section
below.

## Shared conventions

Across every tool in this repo:

- Keep the code simple and readable — clear names, obvious flow.
- Don't over-engineer. No config files, no plugin systems, no abstractions ahead
  of need.
- Written text (comments, docs) follows the ghost-writer voice: direct, simple,
  honest, no fluff.
- When the user pastes full-page content to store — in `knowledge/`, a skill, or
  anywhere else — copy it verbatim. Do not edit, improve, reword, reorganize, or
  clean it up. This applies no matter what the content is or where it's going.
  Copy-paste noise that comes with the page (site artifacts, HTML escapes, and
  the like) stays too; point it out, but only remove it if the user explicitly
  asks.
- When someone asks to improve a tool, push it toward the goal it already
  serves. Extend what's there; don't bolt on machinery.

## Task: build an llms.txt

A user shows up here wanting an `llms.txt` for some library — one clean markdown
file an LLM can read end to end.

1. **They bring the docs.** Don't go fetch anything. The user clones the
   library's repo into this directory and points you at where the docs live
   inside it. Say it's `view_component_repo`, with docs under `docs/guide/`
   plus `docs/best_practices.md`. If they haven't said where, ask.

2. **Run the script.** `bin/frontmatter_concat` cuts the YAML front matter off
   each file and concatenates the rest:

   ```console
   mkdir -p llms_txt/view_component
   bin/frontmatter_concat view_component_repo/docs/guide/*.md \
     view_component_repo/docs/best_practices.md > llms_txt/view_component/llms.txt
   ```

3. **Default output is `llms_txt/<library_name>/llms.txt`.** The name is the
   library's, not the clone folder's — `view_component_repo` becomes
   `view_component`.

### Why we join the files blindly

The result is for an LLM, so it doesn't have to be symmetrical. An LLM reads
markdown with uneven heading levels, different section shapes, and no stitching
between docs without complaint. So we concatenate blindly: no separators, no
normalizing, no reordering. The only thing worth getting right is cutting each
file's header the same way, and that's what the script does.

Each doc keeps its `# Title` after the cut. That title is the only place the
doc's name survives once the front matter is gone, and it gives the combined
file a natural boundary between docs.

### When the format differs

The script cuts Jekyll-style front matter: a `---` block at the very top of a
file. If a library uses a different doc format, don't shoehorn it in. Extend the
script to cut that format instead, and keep the cut narrow and obvious — it
should only ever touch the header.

## Task: serve an llms.txt

`bin/serve_llms_txt` serves `llms_txt/` on `http://localhost:9999`, bound to
this machine only, so an LLM can fetch what you've generated:

```console
bin/serve_llms_txt
```

Then `http://localhost:9999/view_component/llms.txt`. Set `PORT` to move it off
9999. It's Caddy's file server under the hood, so Caddy needs to be installed.

## Task: compose an agent file

Conventions live in `knowledge/` — see `knowledge/index.md` for what's
available. Each file is a bank of sections the user wrote by hand. When the
user asks for an agent file for some language or stack, compose it from these
files. First read `knowledge/caveats.md` — it holds context warnings about the
knowledge files. If a caveat applies to the user's target project, surface it
to the user and let them decide; never resolve or work around it yourself.

**Fresh compose.** Read `knowledge/<lang>.md` and copy its sections into the
new file verbatim. Do not reword, reorganize, or add opinions of your own. If
the user narrows the scope ("only pick XYZ"), include only the sections that
match — still copied verbatim. If the user names a destination, write there.
Otherwise write to `staging/` at the repo root and let the user move the file.

**Blending into an existing file.** If the user points at an existing agent
file, read it and append only the conventions that are missing. Never rewrite,
reorder, or improve what's already there — change as little as possible. If an
existing rule contradicts a convention, don't resolve it: list the conflict
and let the user decide.

**Skill output.** If the user asks for a skill instead, wrap the same content
in skill format rather than a plain agent file. Read
`knowledge/agent-skills/specification.md` for the skill format and
`knowledge/agent-skills/best-practices.md` for how to write good skill
content. Follow both. The content rules above don't change.

## Leave README.md alone

README.md is unfinished. Don't read it — it isn't part of this workflow.
