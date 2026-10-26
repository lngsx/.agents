# Knowledge files stay as pasted; corrections live in caveats.md

Copies in `knowledge/` are stored exactly as they arrived — pasted pages keep
their noise (flagged, never silently cleaned), hand-written banks are never
reworded or reorganized. When a copy carries context that doesn't hold in
every project it may be composed into (e.g. ruby.md says "This project uses
ruby-lsp"), the fix goes into `knowledge/caveats.md`, and the agent surfaces
the caveat when composing — it never resolves or works around one itself.

## Considered Options

- Editing copies in place to remove or generalize project-specific lines —
  rejected: the copy stops matching its source, so mistakes made while
  "cleaning" hide themselves; there is no baseline to diff against, and what
  looks like noise may be content (a literal `&lt;` inside a code sample).
- Auto-stripping obvious mechanical noise on paste — rejected: it makes the
  noise/content judgment silently, inside the agent.
- Annotating copies with inline comments — rejected: that is editing the copy.

The cleanup-vs-preserve judgment is always confirmed by the user with the
exact substitutions shown first. Storage stays lossless; cleanup is a
separate, explicit step.
