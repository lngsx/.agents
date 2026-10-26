# caveats

Context warnings for knowledge files. The copies stay untouched; the warnings
live here. Check before composing from these files.

## Low-visibility sources

Some knowledge files come from small personal repos, not official docs.
Provenance is unverified and unknown. The trade is simple: it was tried on a
real project, and it worked. Version numbers, issue numbers, source line
refs — the author's word, nobody has checked them against upstream. Pick
them up when there's time and make it right.

Files affected:

- `ruby-rails/turbo.md` — from
  https://github.com/fmanimashaun/claude-skills. Low visibility: 1 star,
  4,187+ commits as of Oct 2026. Worked in a real Rails project; whether the
  claims trace to official Rails/Hotwire docs — unknown. The source repo has
  sibling files (`references/production.md`, `stream-security.md`,
  `SKILL.md`) that didn't come along. The file pointed at them; the pointers
  got pulled out before it was banked. Paste-verbatim normally applies —
  this is the exception, and it was the owner's call.

### Safe-to-follow review (Oct 2026)

Read end to end, not checked against official docs. Verdict: safe to
follow. Nothing in it asks for risky behavior. The one security-relevant
passage — `turbo_stream_from` signing — errs conservative: the signature is
integrity, not access control. The unverified specifics are low-stakes too.
A stale version number or wrong issue ref costs a worn fact. Nothing hangs
on them that could hurt.

### ruby-rails/turbo.md — someone else's project in §8b

Section 8b ("A failed request — what the person sees") is written against
the source author's own component library — `Ui::Alert`,
`Ui::AlertComponent`, `Ui::EmptyStateComponent`, "the design-system Toast
entry" — plus a pointer into their project log (`Retask #258`). Compose the
approach, never the component names. Rest of §8b: same treatment as
ruby.md's ruby-lsp note — surface the project-specific bits when composing.

## ruby-rails/ruby.md

- "This project uses ruby-lsp" is specific to the project it was written in.
  Composed agent files for other projects may not use ruby-lsp — surface this
  when composing.
