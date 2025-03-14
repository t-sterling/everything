# 🖥️ Bash, Bash Scripting & Vim: Concepts & Cheatsheet

This guide provides **a deep dive into Bash, Bash scripting, and Vim**, including **essential commands, scripting techniques, and productivity tips**.

📌 **Bash Official Docs**: [GNU Bash Manual](https://www.gnu.org/software/bash/manual/)  
📌 **Vim Official Docs**: [Vim Documentation](https://vimhelp.org/)  

---

## **1. Bash Shell: Core Concepts**  

Bash (Bourne Again Shell) is **a command-line interpreter** for Unix/Linux systems, allowing users to **execute commands, automate tasks, and manage files**.

### **1.1 Common Bash Commands**  
| Command | Description |
|---------|------------|
| `pwd` | Print current directory |
| `ls -l` | List files in long format |
| `cd /path` | Change directory |
| `cp file1 file2` | Copy files |
| `mv file1 file2` | Move/rename files |
| `rm file` | Delete file |
| `find . -name "*.log"` | Search for files |
| `grep "error" log.txt` | Search inside files |

### **1.2 File Permissions in Bash**  
| Command | Description |
|---------|------------|
| `chmod 755 file` | Set execute permissions |
| `chown user:group file` | Change file owner |
| `ls -l` | Show file permissions |

🔗 **More on Bash Commands**: [GNU Bash Reference](https://www.gnu.org/software/bash/manual/)  

---

## **2. Bash Scripting: Automating Tasks**  

Bash scripting enables **automating repetitive tasks** using shell scripts (`.sh` files).

### **2.1 Writing a Basic Script**  
```bash
#!/bin/bash
echo "Hello, Bash!"
```

### **2.2 Variables in Bash**  
```bash
NAME="Alice"
echo "Hello $NAME"
```

### **2.3 Conditional Statements**  
```bash
if [ "$NAME" == "Alice" ]; then
  echo "Hello, Alice!"
else
  echo "Hello, Stranger!"
fi
```

### **2.4 Loops in Bash**  
```bash
for i in {1..5}; do
  echo "Number: $i"
done
```

```bash
while true; do
  echo "Press [CTRL+C] to stop.."
  sleep 1
done
```

### **2.5 Functions in Bash**  
```bash
function greet() {
  echo "Hello, $1!"
}
greet "Alice"
```

🔗 **More on Bash Scripting**: [Bash Scripting Guide](https://linuxconfig.org/bash-scripting-tutorial-for-beginners)  

---

## **3. Vim Editor: Essential Commands**  

Vim is **a powerful text editor** commonly used for editing configuration files and coding.

### **3.1 Vim Modes**  
| Mode | Description |
|------|------------|
| **Normal Mode** | Default mode (navigate, copy, delete, etc.) |
| **Insert Mode** | Enter text (press `i` to switch) |
| **Command Mode** | Save, exit, search (`:` commands) |

### **3.2 Essential Vim Commands**  
| Command | Description |
|---------|------------|
| `i` | Insert mode (start typing) |
| `Esc` | Exit to normal mode |
| `:w` | Save file |
| `:q` | Quit Vim |
| `:wq` or `ZZ` | Save and quit |
| `:q!` | Quit without saving |
| `/search_term` | Search for a word |
| `n` | Next search result |
| `yy` | Copy a line |
| `dd` | Delete a line |
| `p` | Paste copied text |
| `u` | Undo last change |
| `Ctrl + r` | Redo last undo |

🔗 **More on Vim**: [Vim Cheat Sheet](https://vim.rtorr.com/)  

---

### **4. Advanced Bash & Vim Tips**  

#### **4.1 Advanced Bash One-Liners**  
```bash
find . -type f -exec grep "error" {} +  # Find all files containing "error"
ls -lh | awk '{print $9, $5}'  # List files with sizes
```

#### **4.2 Efficient Vim Navigation**  
| Shortcut | Action |
|----------|--------|
| `0` | Move to beginning of line |
| `^` | Move to first non-blank character |
| `$` | Move to end of line |
| `gg` | Jump to first line |
| `G` | Jump to last line |
| `Ctrl + d` | Scroll down half page |
| `Ctrl + u` | Scroll up half page |

🔗 **More on Bash & Vim Tricks**: [Bash One-Liners](https://www.commandlinefu.com/) | [Vim Advanced Guide](https://vimhelp.org/)  

---

### **Final Thoughts**  
Bash and Vim are **essential tools** for system administration, scripting, and development. By mastering **Bash scripting** and **Vim commands**, you can **boost productivity and automate complex tasks**.

### **Happy Scripting & Editing! 🖥️🚀**  
