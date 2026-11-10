# Snapshot and Refs

Compact element references that reduce context usage dramatically for AI agents.

**Related**: [commands.md](commands.md) for full command reference, [SKILL.md](../SKILL.md) for quick start.

## Contents

- [How Refs Work](#how-refs-work)
- [Snapshot Command](#the-snapshot-command)
- [Using Refs](#using-refs)
- [Ref Lifecycle](#ref-lifecycle)
- [Best Practices](#best-practices)
- [Ref Notation Details](#ref-notation-details)
- [Troubleshooting](#troubleshooting)

## How Refs Work

Traditional approach:
```
Full DOM/HTML → AI parses → CSS selector → Action (~3000-5000 tokens)
```

monavi browser approach:
```
Compact snapshot → @refs assigned → Direct interaction (~200-400 tokens)
```

## The Snapshot Command

```bash
# Basic snapshot (shows page structure)
monavi browser snapshot

# Interactive snapshot (-i flag) - RECOMMENDED
monavi browser snapshot -i
```

Compact snapshots (`-c`) keep visible text, status text, and their direct ancestors. Empty sibling branches are removed. Interactive snapshots (`-i`) filter out noninteractive content. MCP snapshots default to interactive; set `interactive: false` to retain visible text and status text.

### Snapshot Output Format

```
Page: Example Site - Home
URL: https://example.com

@e1 [header]
  @e2 [nav]
    @e3 [a] "Home"
    @e4 [a] "Products"
    @e5 [a] "About"
  @e6 [button] "Sign In"

@e7 [main]
  @e8 [h1] "Welcome"
  @e9 [form]
    @e10 [input type="email"] placeholder="Email"
    @e11 [input type="password"] placeholder="Password"
    @e12 [button type="submit"] "Log In"

@e13 [footer]
  @e14 [a] "Privacy Policy"
```

## Using Refs

Once you have refs, interact directly:

```bash
# Click the "Sign In" button
monavi browser click @e6

# Fill email input
monavi browser fill @e10 "user@example.com"

# Fill password
monavi browser fill @e11 "password123"

# Submit the form
monavi browser click @e12
```

## Ref Lifecycle

Same-document updates preserve refs for surviving DOM elements. Replaced elements and page or iframe document replacements invalidate the corresponding refs. Invalidated IDs are never recycled within the browser session. JSON snapshot responses list refs that disappeared in `removedRefs`. Virtual accessibility nodes have snapshot-local refs.

A ref keeps pointing at its DOM node while that node exists. If the node was replaced, monavi browser falls back to the one element with the same role and name, but only when the ref was unique in its snapshot. When duplicates make the match ambiguous, the action fails without acting. Run `snapshot` again and use a fresh ref.

```bash
# Get initial snapshot
monavi browser snapshot -i
# @e1 [button] "Next"

# Click triggers page change
monavi browser click @e1

# Re-snapshot after navigation to get new refs
monavi browser snapshot -i
# @e2 [heading] "Page 2"  ← New element, new ref
```

## Best Practices

### 1. Always Snapshot Before Interacting

```bash
# CORRECT
monavi browser open https://example.com
monavi browser snapshot -i          # Get refs first
monavi browser click @e1            # Use ref

# WRONG
monavi browser open https://example.com
monavi browser click @e1            # Ref doesn't exist yet!
```

### 2. Re-Snapshot After Navigation

```bash
monavi browser click @e5            # Navigates to new page
monavi browser snapshot -i          # Get new refs
monavi browser click @e1            # Use new refs
```

### 3. Re-Snapshot After Dynamic Changes

```bash
monavi browser click @e1            # Opens dropdown
monavi browser snapshot -i          # See dropdown items
monavi browser click @e7            # Select item
```

### 4. Snapshot Specific Regions

For complex pages, snapshot specific areas:

```bash
# Snapshot just the form
monavi browser snapshot @e9
```

## Ref Notation Details

```
@e1 [tag type="value"] "text content" placeholder="hint"
│    │   │             │               │
│    │   │             │               └─ Additional attributes
│    │   │             └─ Visible text
│    │   └─ Key attributes shown
│    └─ HTML tag name
└─ Unique ref ID
```

### Common Patterns

```
@e1 [button] "Submit"                    # Button with text
@e2 [input type="email"]                 # Email input
@e3 [input type="password"]              # Password input
@e4 [a href="/page"] "Link Text"         # Anchor link
@e5 [select]                             # Dropdown
@e6 [textarea] placeholder="Message"     # Text area
@e7 [div class="modal"]                  # Container (when relevant)
@e8 [img alt="Logo"]                     # Image
@e9 [checkbox] checked                   # Checked checkbox
@e10 [radio] selected                    # Selected radio
```

## Iframes

Presentational iframes use the same `Iframe` role for snapshots and stale-ref recovery.

Snapshots automatically detect and inline iframe content. When the main-frame snapshot runs, each `Iframe` node is resolved and its child accessibility tree is included directly beneath it in the output. Refs assigned to elements inside iframes carry frame context, so interactions like `click`, `fill`, and `type` work without manually switching frames.

```bash
monavi browser snapshot -i
# @e1 [heading] "Checkout"
# @e2 [Iframe] "payment-frame"
#   @e3 [input] "Card number"
#   @e4 [input] "Expiry"
#   @e5 [button] "Pay"
# @e6 [button] "Cancel"

# Interact with iframe elements directly using their refs
monavi browser fill @e3 "4111111111111111"
monavi browser fill @e4 "12/28"
monavi browser click @e5
```

**Key details:**
- Only one level of iframe nesting is expanded (iframes within iframes are not recursed)
- Cross-origin iframes that block accessibility tree access are silently skipped
- Empty iframes or iframes with no interactive content are omitted from the output
- To scope a snapshot to a single iframe, use `frame @ref` then `snapshot -i`

## Troubleshooting

### "Ref not found" Error

```bash
# Ref may have changed - re-snapshot
monavi browser snapshot -i
```

### Element Not Visible in Snapshot

```bash
# Scroll down to reveal element
monavi browser scroll down 1000
monavi browser snapshot -i

# Or wait for dynamic content
monavi browser wait 1000
monavi browser snapshot -i
```

### Too Many Elements

```bash
# Snapshot specific container
monavi browser snapshot @e5

# Or use get text for content-only extraction
monavi browser get text @e5
```
