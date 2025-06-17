# [VS code](https://code.visualstudio.com/docs/configure/keybindings)

- `Ctrl b` - toggle primary side bar (overwritten by vim in visual mode)
- `Ctrl 1` - Focus Editor(easy to switch from terminal)
- `Ctrl Shift 5` - Split Terminal(will be in same group)
- `Ctrl PageUp/Down` - Focus Next terminal
- `Alt Up/Down` - Focus Next terminal in same group

> To filter the list to only show the shortcuts you have modified, select the Show User Keybindings command in the More Actions (...) menu. This will show the custom shortcuts defined in `keybindings.json` file.

- `Ctrl Alt/Shift -` - Go Back/Forward
- `Alt z -` - Toggle word wrap
- `Fold/Unfold` - Lookup yourself

[Official Keyboard reference sheet for Linux](https://code.visualstudio.com/shortcuts/keyboard-shortcuts-linux.pdf)



# Bash
![alt text](./Images/image3.png)

| Shortcut          | Description                                  |
|-------------------|----------------------------------------------|
| `Ctrl _`                |   Undo Typing|
| `Not possible`      | Redo Typing |




# [Vim](https://vim.rtorr.com/)

## **Normal Mode**
| Shortcut          | Description                                  |
|-------------------|----------------------------------------------|
| `e` / `ge`/ `gE` / `E`        | Jump to end of next/previous word            |
| `0` / `$`         | Jump to start/end of line                    |
| `^`               | Jump to first non-blank character of line    |
| `gg` / `G`        | Go to first/last line of file                |
| `:n`              | Go to line `n` (e.g., `:10` → line 10)       |
| `}` / `{`         | Next/previous empty line (A paragraph begins after each empty line)                     |
| `H` / `M` / `L`   | Top/Middle/Bottom of screen                  |
| `zt` / `zz` / `zb`| Scroll line to Top/Center/Bottom of screen   |
| `;` / `,`         | Repeat last `f`, `t`, `F`, or `T` (forward/backward) |
| `<C-u>` / `<C-d>` | Half-page up/down                            |
| `<C-b>` / `<C-f>` | Page up/down                                 |
| `<C-e>` / `<C-y>` | Scroll screen up/down by 1 line              |

---

## **Insert Mode**
| Shortcut      | Description                                  |
|--------------|----------------------------------------------|
| `I` / `A`    | Insert at start/end of line                  |
| `o` / `O`    | Insert new line below/above                  |
| `Ctrl + t`   | Indent line right (insert mode)              |
| `Ctrl + d`   | De-indent line left (insert mode)            |
| `s` / `S`      | Delete character/line and enter Insert mode   |
| `C` / `cc`      | Change from cursor to end of line / entire line        |
| `gi`     | Insert at last insertion point           |

---

## **Editing**
| Shortcut      | Description                                  |
|--------------|----------------------------------------------|
| `x` / `X`    | Delete character under/before cursor         |
| `dw` / `dd`  | Delete word/line                             |
| `D`          | Delete from cursor to end of line            |
| `yw` / `yy`  | Yank (copy) word/line                        |
| `p` / `P`    | Paste after/before cursor                    |
| `u` / `Ctrl+r` | Undo/redo                                   |
| `.`          | Repeat last command                          |
| `J` / `gJ`   | Join line below (with/without space)         |
| `>>` / `<<`  | Indent/de-indent line                        |
| `R`          | Replace multiple characters (until `Esc`)    |

---

## **Search & Replace**
| Shortcut        | Description                                  |
|----------------|----------------------------------------------|
| `/pattern`     | Search forward for `pattern`                 |
| `?pattern`     | Search backward for `pattern`                |
| `n` / `N`      | Next/previous match                          |
| `:%s/old/new/g`| Replace all `old` with `new` globally        |

---

## **Navigation & History**
| Shortcut      | Description                                  |
|--------------|----------------------------------------------|
| `Ctrl + i` / `Ctrl + o` | Move forward/backward in jump list       |
| `g;` / `g,`  | Move backward/forward in change list         |
| `:ju[mps]`   | Show jump list                               |
| `:changes`   | Show change list                             |
| `'"`     | Jump to position when last editing file  |
| `'.`     | Jump to position of last change in file  |
| ``` `` ``` | Jump to position before last jump      |
---

## **Text Manipulation**
| Shortcut      | Description                                  |
|--------------|----------------------------------------------|
| `V~` / `g~~` | Toggle case for line                         |
| `VU` / `guu` | Uppercase/lowercase line                     |
| `gggUG`      | Uppercase entire file                        |
| `ggguG`      | Lowercase entire file                        |
| `ggg~G`      | Toggle case entire file                      |

---

### **[Vim Operators](https://quickref.me/vim.html#vim-operators)**
```
[count] <operator> <motion>
<operator> [count] <motion>
```
Example:  
`>4k` → Indent 4 lines upwards

---

### **Clipboard**
| Shortcut          | Description                          |
|------------------|--------------------------------------|
| `"*y` / `"+y`    | Yank to system clipboard             |
| `"*p` / `"+p`    | Paste from system clipboard         |

---
![](https://i.imgur.com/YLInLlY.png)