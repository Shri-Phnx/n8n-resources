# n8n Keyboard Shortcuts — Complete Reference

> **Author:** Shrinivas Ramaprasad  
> **Source:** [n8n Official Docs](https://docs.n8n.io/keyboard-shortcuts/)  
> **Last updated:** April 2026

---

## 1. Workflow Controls

| Shortcut (Windows/Linux) | Shortcut (Mac) | Purpose | Where & How to Use |
|--------------------------|----------------|---------|---------------------|
| `Ctrl + S` | `⌘ Cmd + S` | Save workflow | Anywhere in the workflow editor. Use frequently — n8n does not auto-save in all states. |
| `Ctrl + Z` | `⌘ Cmd + Z` | Undo last action | Canvas or node editor. Reverts node moves, deletions, and edits. |
| `Ctrl + Y` or `Ctrl + Shift + Z` | `⌘ Cmd + Shift + Z` | Redo last undone action | Canvas or node editor. Re-applies the action you just undid. |
| `Ctrl + Enter` | `⌘ Cmd + Enter` | Execute entire workflow | Canvas — no node selected. Runs all nodes from the trigger; same as clicking "Execute workflow". |
| `Ctrl + A` | `⌘ Cmd + A` | Select all nodes | Canvas (click canvas background first). Useful before bulk-copy or bulk-delete. |
| `Ctrl + C` | `⌘ Cmd + C` | Copy selected node(s) | Canvas with nodes selected. Select multiple nodes first with Shift+Click or drag-select. |
| `Ctrl + V` | `⌘ Cmd + V` | Paste node(s) | Canvas. Pasted nodes appear offset from originals — reposition as needed. |
| `Ctrl + X` | `⌘ Cmd + X` | Cut selected node(s) | Canvas with nodes selected. Removes nodes from canvas and copies to clipboard. |

---

## 2. Canvas — Move & Zoom

| Shortcut (Windows/Linux) | Shortcut (Mac) | Purpose | Where & How to Use |
|--------------------------|----------------|---------|---------------------|
| `Scroll wheel` | `Scroll wheel` | Pan canvas vertically | Canvas background. Hold Shift + Scroll to pan horizontally instead. |
| `Shift + Scroll` | `Shift + Scroll` | Pan canvas horizontally | Canvas background. Handy for wide workflows with many parallel branches. |
| `Ctrl + Scroll` | `⌘ Cmd + Scroll` | Zoom in / out | Canvas background. Scroll up to zoom in, scroll down to zoom out. |
| `Ctrl + +` | `⌘ Cmd + +` | Zoom in | Canvas. Also works with the `=` key. |
| `Ctrl + -` | `⌘ Cmd + -` | Zoom out | Canvas. Useful when the canvas gets crowded. |
| `Ctrl + 0` | `⌘ Cmd + 0` | Reset zoom to 100% | Canvas. Returns to the default 1:1 zoom level. |
| `Ctrl + Shift + H` | `⌘ Cmd + Shift + H` | Fit all nodes to screen / tidy up | Canvas. Zooms out to show all nodes; also auto-arranges layout. |
| `Middle mouse drag` | `Middle mouse drag` | Free-pan canvas in any direction | Canvas background. Fastest way to navigate large workflows. |

---

## 3. Nodes on the Canvas

| Shortcut (Windows/Linux) | Shortcut (Mac) | Purpose | Where & How to Use |
|--------------------------|----------------|---------|---------------------|
| `N` | `N` | Open node panel to add a node | Canvas (no input focused). Type immediately to search for a node type. |
| `Tab` | `Tab` | Select next node | Canvas. Cycles forward through nodes in workflow order. |
| `Shift + Tab` | `Shift + Tab` | Select previous node | Canvas. Cycles backward — use Tab/Shift+Tab to navigate without a mouse. |
| `Enter` or Double-click | `Enter` or Double-click | Open / inspect selected node | Canvas with node selected. Opens node settings and output data panel. |
| `F2` | `F2` | Rename selected node | Canvas with node selected. Inline rename — press Enter to confirm, Escape to cancel. |
| `D` | `D` | Disable / enable node | Canvas with node selected. Disabled nodes are skipped during execution (shown with grey overlay). |
| `P` | `P` | Pin / unpin node output data | Canvas with node selected (post-execution). Pinned data persists between runs — useful for testing downstream nodes. |
| `Ctrl + D` | `⌘ Cmd + D` | Duplicate selected node(s) | Canvas with node(s) selected. Duplicate appears adjacent; connections are not copied. |
| `Delete` / `Backspace` | `Delete` / `Backspace` | Delete selected node(s) | Canvas with node(s) selected. Use Ctrl+Z immediately to undo. |
| `Arrow keys` | `Arrow keys` | Nudge selected node(s) by 2px | Canvas with node(s) selected. Fine-tune positioning. |
| `Shift + Arrow keys` | `Shift + Arrow keys` | Move node(s) by 32px | Canvas with node(s) selected. Larger grid jumps for faster repositioning. |
| `Shift + Click` | `Shift + Click` | Add/remove node from selection | Canvas. Build multi-node selections without losing existing ones. |
| `Click + Drag` | `Click + Drag` | Rectangle-select multiple nodes | Canvas background. Drag over empty canvas area to lasso nodes. |

---

## 4. Node Panel

| Shortcut (Windows/Linux) | Shortcut (Mac) | Purpose | Where & How to Use |
|--------------------------|----------------|---------|---------------------|
| `N` | `N` | Open node panel | Canvas. Pressing N focuses the search field — start typing immediately. |
| `↑ / ↓` | `↑ / ↓` | Navigate up/down through results | Node panel (open). Arrow keys cycle through the filtered node list. |
| `Enter` | `Enter` | Add highlighted node to canvas | Node panel with a node highlighted. Adds and immediately opens the node settings. |
| `Tab` | `Tab` | Move to next category / section | Node panel. Quickly jump between trigger types and action categories. |
| `Escape` | `Escape` | Close node panel | Node panel (open). Returns focus to canvas. |

---

## 5. Code Node Editor

| Shortcut (Windows/Linux) | Shortcut (Mac) | Purpose | Where & How to Use |
|--------------------------|----------------|---------|---------------------|
| `Ctrl + Enter` | `⌘ Cmd + Enter` | Execute the code node | Inside Code node editor. Tests your JS/Python without closing the node panel. |
| `Ctrl + /` | `⌘ Cmd + /` | Toggle comment / uncomment line | Code node editor. Works on single or multi-line selections. |
| `Ctrl + Z` | `⌘ Cmd + Z` | Undo in code editor | Code node editor. Separate undo history from the canvas. |
| `Ctrl + F` | `⌘ Cmd + F` | Find in code | Code node editor. Opens inline find/replace within the code block. |
| `Ctrl + G` | `⌘ Cmd + G` | Find next match | Code node editor, after Ctrl+F. Cycles through all search matches. |
| `Alt + Click` | `⌥ Opt + Click` | Add multiple cursors | Code node editor. Edit the same text at multiple positions simultaneously. |
| `Ctrl + D` | `⌘ Cmd + D` | Select next occurrence of selection | Code node editor. Repeated press extends multi-cursor selection. |
| `Alt + ↑ / ↓` | `⌥ Opt + ↑ / ↓` | Move line up / down | Code node editor. Reorders lines without cut-paste. |
| `Ctrl + Shift + K` | `⌘ Cmd + Shift + K` | Delete current line | Code node editor. Removes the whole line in one keystroke. |
| `Tab` | `Tab` | Indent selected code | Code node editor with selection. Shift+Tab to de-indent. |
| `Escape` | `Escape` | Close code editor / return to canvas | Code node editor. Saves changes automatically before closing. |

---

## 6. Command Bar

| Shortcut (Windows/Linux) | Shortcut (Mac) | Purpose | Where & How to Use |
|--------------------------|----------------|---------|---------------------|
| `Ctrl + K` | `⌘ Cmd + K` | Open command bar | Anywhere in n8n. Central launcher — search workflows, nodes, settings, and actions. Commands adapt based on your current view and permissions. |
| `↑ / ↓` | `↑ / ↓` | Navigate command suggestions | Command bar (open). Use arrows to preview different commands before executing. |
| `Enter` | `Enter` | Execute selected command | Command bar (open). Instantly jumps to the selected workflow, node, or action. |
| `Escape` | `Escape` | Close command bar | Command bar (open). Returns to previous view without executing anything. |

---

## Quick Summary Card

| Action | Windows/Linux | Mac |
|--------|--------------|-----|
| Save | `Ctrl+S` | `⌘+S` |
| Undo / Redo | `Ctrl+Z` / `Ctrl+Y` | `⌘+Z` / `⌘+Shift+Z` |
| Execute workflow | `Ctrl+Enter` | `⌘+Enter` |
| Add node | `N` | `N` |
| Rename node | `F2` | `F2` |
| Disable node | `D` | `D` |
| Pin data | `P` | `P` |
| Duplicate node | `Ctrl+D` | `⌘+D` |
| Delete node | `Delete` | `Delete` |
| Select all | `Ctrl+A` | `⌘+A` |
| Copy / Paste | `Ctrl+C` / `Ctrl+V` | `⌘+C` / `⌘+V` |
| Fit to screen | `Ctrl+Shift+H` | `⌘+Shift+H` |
| Command bar | `Ctrl+K` | `⌘+K` |
| Execute code node | `Ctrl+Enter` | `⌘+Enter` |

---

*Reference: [n8n Keyboard Shortcuts Docs](https://docs.n8n.io/keyboard-shortcuts/) · [n8n Code Editor Shortcuts](https://docs.n8n.io/integrations/builtin/core-nodes/n8n-nodes-base.code/keyboard-shortcuts/)*
