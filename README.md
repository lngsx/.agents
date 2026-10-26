This repo holds my agent tooling and conventions.

- **Build an llms.txt.** Concatenates a cloned library's docs into one file: `llms_txt/<library>/llms.txt`. Cloned repos (`*repo*/`) are inputs, not part of this repo — gitignored.
- **Serve llms.txt.** Serves `llms_txt/` on `localhost:9999` so my agent can fetch it.
- **Compose an agent file.** Creates a new agent file from sections in `knowledge/`, into `staging/` for review and do hand copy to the place I want to use.
- **Blend into an existing file.** Appends only the missing conventions to an existing agent file, and flags conflicts instead of resolving them.
- **Compose a skill.** Same content, wrapped in skill format, per `knowledge/agent-skills/`.
- **Store a page.** Copies pasted content exactly as-is — no cleanup, no rewording. Noise gets flagged, not silently stripped.

How the pieces connect:

```
pasted pages ──> knowledge/            (source of truth)
                    |
                    | sections, copied as-is
                    |
        +-----------+------------+
        v           v            v
   new agent     existing     skill
   file          agent file   (SKILL.md format)
   (staging/)    (append only)
```

The llms.txt pipeline is separate — it draws from cloned repos, not `knowledge/`:

```
cloned repo docs
       |
       v
llms_txt/<library>/llms.txt
       |
       v
localhost:9999  (my agent fetches it)
```

`knowledge/` is mostly copies of real docs from the internet, plus my own conventions — the source of truth for what goes into agent files. `knowledge/index.md` lists what's there; if it's not on the index, it doesn't exist, so add a file and a row first.

Full instructions live in `AGENTS.md`.
