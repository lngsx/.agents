# NAVIGATION.md
# Project navigation and file map for agent orientation.
# Format: YAML with usage instructions as header comments.

# ── How to use this file ──
# Before reading a file, check its entry here. The annotations often answer
# your question without opening the file.
#
# Each file entry is a four-item sequence:
# [path, description, token_density, annotations]
#
# - path, description, token_density: mandatory positional prefix, in that order.
# - annotations: fourth item, a keyed mapping. purpose is required.
# - token_density: light = safe to read directly. dense = use the active file-examine route or read selectively.
# - purpose: why the file exists. Use this to decide if it's relevant.
# - key_concepts: what you'd learn by reading it. Use this to orient without reading.
# - key_exports: what it exposes — interfaces, commands, outputs.
# - reads_from: dependencies. Use this to trace relationships.

files:
  - - AGENTS.md
    - Agent instructions: shared conventions plus per-task sections
    - dense
    - purpose: The rulebook agents follow in this repo. Defines shared conventions (ghost-writer voice, as-is paste rule) and per-task sections — build/serve llms.txt, compose agent files and skills.
      key_concepts:
        - compose task (fresh compose, blend, skill output)
        - as-is paste convention
        - caveats-first reading order
        - frontmatter stripping for llms.txt
      key_exports:
        - "Task: build an llms.txt"
        - "Task: serve an llms.txt"
        - "Task: compose an agent file"
      reads_from:
        - knowledge/index.md
        - knowledge/caveats.md
        - knowledge/agent-skills/specification.md
        - knowledge/agent-skills/best-practices.md

  - - README.md
    - Feature overview with two pipeline diagrams
    - light
    - purpose: 30-second orientation for a human. Six features, one chronological list, plus the knowledge-flow and llms.txt-flow diagrams. Not content — points at AGENTS.md for instructions.
      key_concepts:
        - llms.txt pipeline (cloned repo docs → llms_txt/ → localhost:9999)
        - knowledge flow (pasted pages → knowledge/ → agent file / existing file / skill)

  - - knowledge/index.md
    - Single listing of everything in knowledge/
    - light
    - purpose: The only place that enumerates knowledge files. AGENTS.md and README defer to it. New knowledge file means adding one row here and nothing else.
      key_concepts:
        - index-first rule ("if it's not on the index, it doesn't exist")
      reads_from:
        - knowledge/ruby.md
        - knowledge/rust.md
        - knowledge/typescript.md
        - knowledge/agent-skills/specification.md
        - knowledge/agent-skills/best-practices.md
        - knowledge/caveats.md

  - - knowledge/ruby.md
    - Hand-written Ruby conventions
    - dense
    - purpose: The user's own Ruby/Rails style rules, copied from the old README with content preserved. Read when composing a Ruby agent file.
      key_concepts:
        - 2-space formatting, method organization
        - functional style (map, filter_map, then)
        - modern Ruby (pattern matching, endless methods)
        - dry-struct over Struct
        - RBS inline annotations over YARD
      reads_from:
        - knowledge/caveats.md

  - - knowledge/rust.md
    - Empty stub for future Rust conventions
    - light
    - purpose: Placeholder; recreate only when content exists.

  - - knowledge/typescript.md
    - Empty stub for future TypeScript conventions
    - light
    - purpose: Placeholder; recreate only when content exists.

  - - knowledge/caveats.md
    - Context warnings about knowledge files
    - light
    - purpose: Holds warnings for copied content that doesn't hold in every project (e.g. ruby.md assumes ruby-lsp). The agent reads this before composing and surfaces caveats instead of resolving them. The copies themselves stay untouched.
      key_concepts:
        - annotation-not-modification
        - flag-don't-resolve discipline

  - - knowledge/agent-skills/specification.md
    - Agent Skills format spec (copied from agentskills.io)
    - dense
    - purpose: Exact copy of the skill format specification. Read when composing or validating a skill (SKILL.md frontmatter, directory layout, progressive disclosure).
      key_concepts:
        - SKILL.md frontmatter (name, description, compatibility, allowed-tools)
        - progressive disclosure (metadata → instructions → resources)
        - scripts/ references/ assets/ conventions

  - - knowledge/agent-skills/best-practices.md
    - Skill-writing best practices (copied from agentskills.io)
    - dense
    - purpose: Exact copy of how to write well-scoped skill content — grounding in real expertise, context budgeting, gotchas sections, validation loops. Read alongside the spec when composing a skill.
      key_concepts:
        - start from real expertise
        - progressive disclosure / context budget
        - gotchas sections
        - plan-validate-execute
        - defaults, not menus

  - - bin/frontmatter_concat
    - Concatenates docs with YAML front matter stripped
    - light
    - purpose: Builds an llms.txt by cutting Jekyll-style front matter off each input file and concatenating the rest, no separators. Output convention is llms_txt/<library_name>/llms.txt (library name, not clone folder name).
      key_concepts:
        - front matter cut (only the header, always the same way)
        - blind concatenation (LLM-tolerant of uneven headings)

  - - bin/serve_llms_txt
    - Serves llms_txt/ on localhost:9999 via a Dockerized Caddy
    - light
    - purpose: Local file server so an agent can fetch generated llms.txt files. Runs Caddy in a detached Docker container (--restart unless-stopped) so it survives reboots; bind to 127.0.0.1 only; PORT env moves it off 9999. Requires Docker installed.
      key_concepts:
        - localhost-only binding
        - containerized Caddy (--restart unless-stopped)
        - llms_txt/ mounted read-only into the container
      reads_from:
        - llms_txt/

  - - docs/adr/0001-knowledge-stays-as-pasted.md
    - ADR: knowledge files stay as pasted; corrections live in caveats.md
    - light
    - purpose: Records why copied content is never edited in place and why the fix path is caveats.md, not the knowledge file.
      key_concepts:
        - as-is storage
        - caveats as annotation layer
