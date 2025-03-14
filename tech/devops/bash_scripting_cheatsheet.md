# 🖥️ Extensive Bash Scripting Cheat Sheet

This cheat sheet provides **essential and advanced Bash scripting techniques**, covering **variables, conditionals, loops, functions, file handling, and debugging**.

📌 **Bash Official Docs**: [GNU Bash Manual](https://www.gnu.org/software/bash/manual/)  
📌 **Bash Scripting Guide**: [Bash Scripting Tutorial](https://linuxconfig.org/bash-scripting-tutorial-for-beginners)  

---

## **1. Basic Script Structure**  

```bash
#!/bin/bash  # Shebang (specifies interpreter)

# Print text
echo "Hello, World!"

# Exit script with success status
exit 0
```

---

## **2. Variables & Environment**  

### **2.1 Declaring Variables**  
```bash
NAME="Alice"
AGE=30
echo "Hello, $NAME! You are $AGE years old."
```

### **2.2 Special Variables**  
| Variable | Description |
|----------|------------|
| `$0` | Script name |
| `$1, $2, ...` | Positional arguments |
| `$#` | Number of arguments |
| `$@` | All arguments as list |
| `$?` | Exit status of last command |
| `$$` | Process ID of script |
| `$USER` | Current username |
| `$HOME` | User home directory |

```bash
echo "Script name: $0"
echo "First argument: $1"
echo "Number of arguments: $#"
```

---

## **3. Conditional Statements**  

### **3.1 If-Else Condition**  
```bash
if [ "$NAME" == "Alice" ]; then
  echo "Hello, Alice!"
elif [ "$NAME" == "Bob" ]; then
  echo "Hello, Bob!"
else
  echo "Hello, Stranger!"
fi
```

### **3.2 Numeric Comparisons**  
| Operator | Meaning |
|----------|---------|
| `-eq` | Equal |
| `-ne` | Not equal |
| `-gt` | Greater than |
| `-lt` | Less than |
| `-ge` | Greater than or equal |
| `-le` | Less than or equal |

```bash
if [ "$AGE" -ge 18 ]; then
  echo "You are an adult."
fi
```

### **3.3 String Comparisons**  
| Operator | Meaning |
|----------|---------|
| `==` | Equals |
| `!=` | Not equal |
| `-z` | String is empty |
| `-n` | String is not empty |

```bash
if [ -z "$NAME" ]; then
  echo "Name is empty."
fi
```

---

## **4. Loops in Bash**  

### **4.1 For Loop**  
```bash
for i in {1..5}; do
  echo "Iteration: $i"
done
```

### **4.2 While Loop**  
```bash
COUNT=0
while [ $COUNT -lt 5 ]; do
  echo "Count: $COUNT"
  ((COUNT++))
done
```

### **4.3 Until Loop**  
```bash
NUM=10
until [ $NUM -lt 5 ]; do
  echo "Number: $NUM"
  ((NUM--))
done
```

---

## **5. Functions in Bash**  

### **5.1 Defining and Calling Functions**  
```bash
function greet() {
  echo "Hello, $1!"
}
greet "Alice"
```

### **5.2 Return Values**  
```bash
function add() {
  echo $(($1 + $2))
}
RESULT=$(add 5 10)
echo "Sum: $RESULT"
```

---

## **6. File Handling & Redirection**  

### **6.1 Reading a File Line by Line**  
```bash
while read LINE; do
  echo "$LINE"
done < file.txt
```

### **6.2 Writing to a File**  
```bash
echo "Hello, Bash!" > output.txt
```

### **6.3 Appending to a File**  
```bash
echo "New line added." >> output.txt
```

### **6.4 Redirecting Output**  
| Operator | Purpose |
|----------|---------|
| `>` | Redirect output (overwrite) |
| `>>` | Redirect output (append) |
| `2>` | Redirect stderr |
| `&>` | Redirect stdout & stderr |

```bash
ls > filelist.txt  # Save output to file
ls 2> errors.log   # Save errors only
ls &> all_output.log  # Save all output
```

---

## **7. Process Management**  

### **7.1 Running Processes in Background**  
```bash
./script.sh &  # Run in background
jobs  # List background jobs
```

### **7.2 Killing Processes**  
```bash
ps aux | grep myprocess
kill -9 PID
```

---

## **8. Debugging Bash Scripts**  

### **8.1 Enable Debug Mode**  
```bash
bash -x script.sh  # Debug entire script
```

### **8.2 Debug Specific Sections**  
```bash
set -x  # Enable debugging
echo "Debugging..."
set +x  # Disable debugging
```

---

## **9. Error Handling & Exit Codes**  

### **9.1 Checking Exit Status**  
```bash
mkdir /root/test 2>/dev/null
if [ $? -ne 0 ]; then
  echo "Failed to create directory!"
fi
```

### **9.2 Trap Errors & Cleanup**  
```bash
trap "echo 'Error occurred! Cleaning up...'; exit 1" ERR
```

---

## **10. Advanced Bash Tricks**  

### **10.1 One-Liners**  
```bash
find . -type f -exec grep "error" {} +  # Find files containing "error"
ls -lh | awk '{print $9, $5}'  # List files with sizes
```

### **10.2 Using Arrays in Bash**  
```bash
FRUITS=("Apple" "Banana" "Cherry")
echo "First fruit: ${FRUITS[0]}"
```

### **10.3 Using `case` Statements**  
```bash
case "$1" in
  start) echo "Starting...";;
  stop) echo "Stopping...";;
  restart) echo "Restarting...";;
  *) echo "Usage: $0 {start|stop|restart}";;
esac
```

---

### **Final Thoughts**  
Bash scripting is a **powerful way to automate tasks**, manipulate files, and manage system processes. By mastering **variables, loops, conditionals, functions, and debugging techniques**, you can **write robust automation scripts** for any Linux system.

### **Happy Scripting! 🖥️🚀**  
