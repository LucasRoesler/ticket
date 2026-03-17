# Ticket Workflow Guide

Tickets are markdown files stored in `TICKETS_DIR` (your Obsidian vault at
`~/Documents/obsidian/notes/tasks/tickets`). Because `TICKETS_DIR` is set
globally, all list commands automatically filter to the current project via
the `ticket project-id` plugin — no `-T` flag needed.

Partial ID matching works everywhere: `tk show 5c4` matches `proj-5c46`.

---

## 1. Find the next task(s)

```bash
# What can I work on right now? (open/in-progress, all deps resolved)
tk ready

# What is blocked and why?
tk blocked

# Show the dependency tree for a specific ticket
tk dep tree <id>
```

`ready` is the primary daily-driver. It sorts by priority (0 = highest) and
only shows tickets whose dependencies are all closed.

---

## 2. Create a ticket

```bash
# Minimal
tk create "Fix login redirect loop"

# With detail
tk create "Fix login redirect loop" \
  -t bug \
  -p 1 \
  -d "Users are redirected in a loop after OAuth callback" \
  --acceptance "Login flow completes without redirect loop" \
  --tags auth,oauth
```

Types: `bug` `feature` `task` `epic` `chore` (default: `task`)
Priority: `0`–`4`, lower is higher (default: `2`)

The command prints the new ticket ID.

---

## 3. Break down a ticket into subtasks

Create child tickets with `--parent`, then express the dependency order with
`dep` so `ready` surfaces them in the right sequence.

```bash
# Create the parent epic
parent=$(tk create "Overhaul auth flow" -t epic)

# Create subtasks under it
t1=$(tk create "Audit existing OAuth callbacks" --parent $parent)
t2=$(tk create "Fix redirect loop" -t bug --parent $parent)
t3=$(tk create "Add integration tests" --parent $parent)

# Express order: t2 depends on t1, t3 depends on t2
tk dep $t2 $t1
tk dep $t3 $t2

# Verify the tree
tk dep tree $parent
```

`tk show <parent>` also lists all children with their statuses.

---

## 4. Set ticket status

```bash
# Start working on it
tk start <id>

# Mark done
tk close <id>

# Reopen if needed
tk reopen <id>
```

---

## 5. Manage ticket content

```bash
# Read a ticket
tk show <id>

# Open in $EDITOR (requires ticket-edit plugin — see below)
tk edit <id>

# Append a timestamped note (inline or piped)
tk add-note <id> "Traced the issue to the session middleware"
echo "See PR #42" | tk add-note <id>
```

### Installing the edit plugin

`ticket-edit` is bundled in the repo but not symlinked yet:

```bash
ln -sf ~/Documents/code/ticket/plugins/ticket-edit ~/.local/bin/ticket-edit
```

Then `tk edit <id>` opens the raw markdown file in `$EDITOR`. The file
contains YAML frontmatter followed by free-form markdown — description,
design notes, acceptance criteria, and any notes appended via `add-note`.

---

## Project ID

Each project gets a stable 8-character ID derived from the git initial commit:

```bash
tk project-id          # print current project's ID
tk project-id set foo  # override with a manual slug (saved to .ticket-project-id)
```

This ID is used automatically as the tag filter when `TICKETS_DIR` is set,
keeping your vault's tickets separated by project.
