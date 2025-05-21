# 

**Department:** Computer Science, School of Engineering  
 **Course:** System Programming  
 **Instructor:** Dr. M. Bilal Qureshi  
 **Class:** CS1-22 / CS2-22 / CS3-22 / CS4-22  
 **Assignment Title:** Theoretical Assignment 1 – Spring 2025  
 **Student:** Ibragimov Bobir  
 **Student ID:** 210095  
 **Submission Date:** March 3, 2025  
 **Total Marks:** 100 (10 questions × 10 points each)

---

## **Question 1: Shell Script Utilizing `grep` with Error Handling via `trap`**

### **Objective:**

Develop a shell script that accepts a filename and a search pattern as arguments. The script should employ `grep` to locate the pattern within the file and implement error handling using `trap` to manage potential execution errors.

### **Rewritten Script:**

bash  
CopyEdit  
`#!/bin/bash`

`# Define custom exit codes`  
`ERR_NO_FILE=65`  
`ERR_NO_PERMISSION=66`  
`ERR_INVALID_PATTERN=67`

`# Set up traps for interruption and termination signals`  
`trap 'echo "Execution interrupted."; exit 1' INT TERM`  
`trap 'echo "Termination signal received."; exit 1' HUP`

`# Verify that two arguments are provided`  
`if [ $# -lt 2 ]; then`  
  `echo "Usage: $0 <filename> <pattern>"`  
  `exit 1`  
`fi`

`file=$1`  
`pattern=$2`

`# Check if the file exists`  
`if [ ! -f "$file" ]; then`  
  `echo "Error: File '$file' not found."`  
  `exit $ERR_NO_FILE`  
`fi`

`# Check if the file is readable`  
`if [ ! -r "$file" ]; then`  
  `echo "Error: Cannot read file '$file'."`  
  `exit $ERR_NO_PERMISSION`  
`fi`

`# Search for the pattern in the file`  
`echo "Searching for '$pattern' in '$file'..."`  
`result=$(grep "$pattern" "$file" 2>&1)`  
`status=$?`

`# Handle grep results`  
`if [ $status -ne 0 ]; then`  
  `if [[ "$result" == *"invalid"* || "$result" == *"syntax error"* ]]; then`  
    `echo "Error: Invalid pattern '$pattern'."`  
    `exit $ERR_INVALID_PATTERN`  
  `fi`  
  `echo "Pattern not found or an error occurred: $result"`  
  `exit 1`  
`else`  
  `echo "Search results:"`  
  `echo "$result"`  
`fi`

`echo "Search completed successfully."`  
`exit 0`

### **Testing Scenarios:**

**Missing Arguments:**

 bash  
CopyEdit  
`./search_pattern.sh`  
 *Output:*

 php-template  
CopyEdit  
`Usage: ./search_pattern.sh <filename> <pattern>`

1. 

**Non-existent File:**

 bash  
CopyEdit  
`./search_pattern.sh nonexistent.txt "search_term"`  
 *Output:*

 javascript  
CopyEdit  
`Error: File 'nonexistent.txt' not found.`

2. 

**Valid File and Pattern:**

 bash  
CopyEdit  
`./search_pattern.sh sample.txt "search_term"`  
 *Output:*

 sql  
CopyEdit  
`Searching for 'search_term' in 'sample.txt'...`  
`Search results:`  
`[Matching lines]`  
`Search completed successfully.`

3. 

**Invalid Pattern:**

 bash  
CopyEdit  
`./search_pattern.sh sample.txt "[a-z"`  
 *Output:*

 javascript  
CopyEdit  
`Searching for '[a-z' in 'sample.txt'...`  
`Error: Invalid pattern '[a-z'.`

4. 

---

## **Question 2: Demonstrating `set`, `unset`, and `shift` Commands in a Shell Script**

### **Objective:**

Create a shell script that showcases the usage of `set`, `unset`, and `shift` commands. The script should accept command-line arguments, assign and remove variables, and shift arguments to illustrate their effects.

### **Rewritten Script:**

bash  
CopyEdit  
`#!/bin/bash`

`echo "Initial command-line arguments:"`  
`echo "Count: $#"`  
`echo "Arguments: $@"`

`# Display first and second arguments if available`  
`if [ $# -ge 1 ]; then`  
  `echo "First argument: $1"`  
`fi`  
`if [ $# -ge 2 ]; then`  
  `echo "Second argument: $2"`  
`fi`

`# Store original arguments`  
`original_args=("$@")`

`echo -e "\nSetting new positional parameters:"`  
`set apple banana cherry date`  
`echo "New arguments: $@"`  
`echo "First: $1"`  
`echo "Second: $2"`  
`echo "Third: $3"`  
`echo "Fourth: $4"`

`echo -e "\nAssigning variables:"`  
`fruit1="mango"`  
`fruit2="orange"`  
`echo "fruit1: $fruit1"`  
`echo "fruit2: $fruit2"`

`echo -e "\nUnsetting 'fruit1':"`  
`unset fruit1`  
`echo "fruit1: ${fruit1:-unset}"`  
`echo "fruit2: $fruit2"`

`echo -e "\nDemonstrating 'shift' command:"`  
`echo "Before shift: $@"`  
`shift`  
`echo "After shifting once: $@"`  
`echo "Current first argument: $1"`

`echo -e "\nShifting two more times:"`  
`shift 2`  
`echo "After shifting twice: $@"`  
`echo "Current first argument: $1"`

`echo -e "\nUsing 'set' with command substitution:"`  
`set $(ls -1 | head -3)`  
`echo "New arguments from 'ls': $@"`

### **Execution Example:**

bash  
CopyEdit  
`./variable_demo.sh arg1 arg2 arg3`

*Sample Output:*

yaml  
CopyEdit  
`Initial command-line arguments:`  
`Count: 3`  
`Arguments: arg1 arg2 arg3`  
`First argument: arg1`  
`Second argument: arg2`

`Setting new positional parameters:`  
`New arguments: apple banana cherry date`  
`First: apple`  
`Second: banana`  
`Third: cherry`  
`Fourth: date`

`Assigning variables:`  
`fruit1: mango`  
`fruit2: orange`

`Unsetting 'fruit1':`  
`fruit1: unset`  
`fruit2: orange`

`Demonstrating 'shift' command:`  
`Before shift: apple banana cherry date`  
`After shifting once: banana cherry date`  
`Current first argument: banana`

`Shifting two more times:`  
`After shifting twice: date`  
`Current first argument: date`

`Using 'set' with command substitution:`  
`New arguments from 'ls': file1.txt file2.txt file3.txt`  
