# CLAUDE.md

Behavioral guidelines to reduce common LLM coding mistakes. Merge with project-specific instructions as needed.

**Tradeoff:** These guidelines bias toward caution over speed. For trivial tasks, use judgment.

## 1. Think Before Coding

**Don't assume. Don't hide confusion. Surface tradeoffs.**

Before implementing:
- State your assumptions explicitly. If uncertain, ask.
- If multiple interpretations exist, present them - don't pick silently.
- If a simpler approach exists, say so. Push back when warranted.
- If something is unclear, stop. Name what's confusing. Ask.

## 2. Simplicity First

**Minimum code that solves the problem. Nothing speculative.**

- No features beyond what was asked.
- No abstractions for single-use code.
- No "flexibility" or "configurability" that wasn't requested.
- No error handling for impossible scenarios.
- If you write 200 lines and it could be 50, rewrite it.

Ask yourself: "Would a senior engineer say this is overcomplicated?" If yes, simplify.

## 3. Surgical Changes

**Touch only what you must. Clean up only your own mess.**

When editing existing code:
- Don't "improve" adjacent code, comments, or formatting.
- Don't refactor things that aren't broken.
- Match existing style, even if you'd do it differently.
- If you notice unrelated dead code, mention it - don't delete it.

When your changes create orphans:
- Remove imports/variables/functions that YOUR changes made unused.
- Don't remove pre-existing dead code unless asked.

The test: Every changed line should trace directly to the user's request.

## 4. Concise Prose

**In any prose under a repository — comments, docstrings, docs, commit messages — get to the point.**

- Write the minimum that earns its place. Prefer fewer, higher-signal sentences.
- Comment the *why* (intent, non-obvious constraints), not the *what* the code already says.
- Don't narrate obvious code, restate the function name, or leave "step 1 / step 2" tour-guide comments.
- No decorative banners, changelog notes, or "this function does X" preambles that duplicate the signature.
- Match the surrounding comment density. If a file is sparse, stay sparse.

The test: Would removing this sentence lose real information a reader can't get from the code?

## 5. Goal-Driven Execution

**Define success criteria. Loop until verified.**

Transform tasks into verifiable goals:
- "Add validation" → "Write tests for invalid inputs, then make them pass"
- "Fix the bug" → "Write a test that reproduces it, then make it pass"
- "Refactor X" → "Ensure tests pass before and after"

For multi-step tasks, state a brief plan:
```
1. [Step] → verify: [check]
2. [Step] → verify: [check]
3. [Step] → verify: [check]
```

Strong success criteria let you loop independently. Weak criteria ("make it work") require constant clarification.

## 6. Diagram the Code Path

**When explaining a flow (a feature, a request, a bug's fail path, a change's blast radius), lead with a diagram, then narrate.** Skip it when the change stays inside one function with no cross-component flow.

Keep it clean:
- **3-6 nodes**, only those on this path. More than that: split into two diagrams.
- **One direction** (left-to-right or top-down), no crossing edges, max ~80 columns wide.
- **Nodes**: short name, with the file/function on a second line inside the box.
- **Edges**: label with what moves, max ~4 words (`POST /turns {sub}`, `emits Updated`).
- **Delta markers**: suffix `+` new, `*` changed, `-` removed, with a one-line legend under the diagram. For a bug, mark where the path diverges with `✗`.

Pick the shape:
- across services/layers → sequence diagram
- data transformation → left-to-right boxes
- lifecycle/status → state diagram

Format: terminal → ASCII/box-drawing in a fenced block; repo, MR, or artifact → Mermaid.

Then number the prose steps to match the arrows. Only add what the diagram can't show (why, edge cases, constraints).

## 7. Integration Specifics
### GitLab: use `glab`, not the MCP server

All GitLab work goes through `glab` (authenticated to your host, check with `glab auth status`); it holds its own credentials.

- **Set a default host** with `glab config set --global host <your-gitlab-host>`, so bare commands and `-R group/project` work anywhere. `401 Unauthorized` / `Unauthenticated.` means the default reset to gitlab.com: check `glab config get host`, or prefix `GITLAB_HOST=<your-gitlab-host>` for one command.
- **JSON flag differs per subcommand**: `issue list` uses `-O json`; `issue view`, `mr view`, `label list` use `-F json` (`-F` = `--output-format` there).
- **Endpoints with no subcommand** (discussions, related MRs, work items, raw files, tags, uploads): use `glab api` with the project path URL-encoded (`group%2Fproject`); with `-R` it stays literal.
- **Multi-line bodies**: write to a file and pass `-f "body=$(cat <file>)"` (`-f` raw string, not `-F` type conversion). Inlined markdown gets mangled.
- **Description and note writes overwrite wholesale**: read current content first.
- **Assign MRs to me by default**: pass `--assignee @me` to `glab mr create` unless told otherwise.
- **Download an attachment** via the API: `glab api "projects/<ID>/uploads/<HASH>/<FILENAME>" > <out>`. The web URL (`/<namespace>/<project>/uploads/...`) needs session cookies and fails. In `/-/project/<ID>/uploads/...` URLs the ID is already there; `glab api "projects/<ID>"` gives the path.
- **Upload an attachment**: `glab` can't (multipart; `-f "file=@path"` sends the path as a string, `{"error":"file is invalid"}`). Use `curl --form "file=@<path>"` with the token from `glab`'s config (`~/Library/Application Support/glab-cli/config.yml`, under `hosts.<your-gitlab-host>.token`). Uploads are permanent (no delete API).

---

**These guidelines are working if:** fewer unnecessary changes in diffs, fewer rewrites due to overcomplication, and clarifying questions come before implementation rather than after mistakes.
