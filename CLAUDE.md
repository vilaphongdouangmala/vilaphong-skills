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

**A diagram always speaks better than paragraphs of text. When explaining how
something flows, show it first, then narrate it.**

Whenever you explain a code path (how a feature works, how a request is handled,
where a bug's fail path runs, or what a proposed change touches), lead with a
diagram of the components involved and how logic/data moves between them. Prose
follows the diagram, not the other way around. If you find yourself writing
three paragraphs to describe a flow, stop and draw it instead; the paragraphs
usually collapse to a numbered list once the picture exists.

What the diagram must do:
- **Scope to the question.** Only the components on this path, not the whole
  system. Aim for 3-8 nodes. If it needs more, split into two diagrams.
- **Label the edges with what moves**, not just that something moves: the
  payload, event, query, or return value (`POST /turns {sub}`, `sub[] -> name[]`,
  `emits ConversationUpdated`).
- **Mark what the change affects.** Tag nodes/edges as `[NEW]`, `[CHANGED]`, or
  `[REMOVED]` so the reader sees the delta against the current flow at a glance.
  For a bug, mark where the path diverges from the expected one.
- **Anchor nodes to code.** Name the file (and function/class where useful) on
  each node so the diagram doubles as a navigation map.
- **Pick the shape that fits the question:**
  - request/response across services or layers -> sequence diagram
  - data transformation pipeline -> left-to-right flow of boxes
  - lifecycle/status transitions -> state diagram

Format:
- Answers in the terminal: ASCII/box-drawing in a fenced code block (Mermaid
  does not render there).
- Text going into the repo, an MR description, or an artifact: Mermaid, since
  GitLab and artifact pages render it.

Then walk the prose in the same order as the diagram, numbering the steps so
each sentence maps to an arrow. Keep that prose short: the diagram carries the
structure, the text only adds what a box or arrow cannot (why, edge cases,
constraints).

Skip the diagram when the change is confined to one function in one file and
there is no cross-component flow to show.

The test: Could a reader who only sees the diagram name the components touched,
what passes between them, and which parts are new or changed? And could you
delete half the prose without losing information the diagram already shows?

---

**These guidelines are working if:** fewer unnecessary changes in diffs, fewer rewrites due to overcomplication, and clarifying questions come before implementation rather than after mistakes.

## 7. Integration Specifics
### GitLab: use `glab`, not the MCP server

All GitLab work goes through the `glab` CLI (authenticated to your host; check with
`glab auth status`). It holds its own credentials, so no token handling is needed.

- **Set a default host** with `glab config set --global host <your-gitlab-host>`, so bare
  commands and `-R group/project` work from anywhere without extra flags. If you ever see
  `401 Unauthorized` or `Unauthenticated.`, the default may have been reset to gitlab.com — check
  `glab config get host` before suspecting the token. Prefix with `GITLAB_HOST=<your-gitlab-host>`
  to override for a single command.
- **The JSON flag is inconsistent between subcommands**: `issue list` uses `-O json`;
  `issue view`, `mr view`, and `label list` use `-F json` (there `-F` means `--output-format`).
- **Endpoints with no subcommand** (discussions, related MRs, work items, raw files, tags,
  uploads) go through `glab api`. URL-encode the project path there (`group%2Fproject`);
  with `-R` it stays literal.
- **Multi-line bodies** (comments, issue descriptions) must be written to a file and passed as
  `-f "body=$(cat <file>)"`. Use `-f` (raw string), not `-F` (magic type conversion). Inlining
  markdown with backticks and newlines as a shell argument mangles it.
- **Writes to descriptions and notes overwrite wholesale** — always read current content first.

### GitLab Attachments

- To download, use the **API endpoint** (not the web URL):
  ```bash
  GITLAB_HOST=<your-gitlab-host> glab api \
    "projects/<PROJECT_ID>/uploads/<HASH>/<FILENAME>" > <OUTPUT_FILE>
  ```
- The web URL format (`/<namespace>/<project>/uploads/...`) does NOT work — it expects session cookies.
- To find the project ID from a URL like `/-/project/<ID>/uploads/...`, the ID is already in the URL. To find the project path, use `glab api "projects/<ID>"`.
- **Uploading** a file is the one case `glab` can't do: uploads are multipart and
  `glab api -f "file=@path"` sends the path as a string (`{"error":"file is invalid"}`). Use
  `curl --form "file=@<path>"` with the token read from `glab`'s config
  (`~/Library/Application Support/glab-cli/config.yml`, under `hosts.<your-gitlab-host>.token`).
  Uploads are permanent; GitLab has no delete-upload API.
