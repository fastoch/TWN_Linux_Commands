# Intro

src = https://www.youtube.com/watch?v=fwP2JW_VnZI

# Scenario 

You're managing a web app that's been running for a while.  
This application generates various log files every day, and you need to investigate error messages that users have been reporting.  

## Pipe

Example: 
```bash
ls -R | grep ".txt"
```
This will return all .txt files in the current folder (`-R` is for Recursive)

- Pipe `|` connects commands together by sending the output of the left command directly as input to the right command
- `grep` = global regular expression print, powerful utility used to search for specific patterns within files or a stream of data

## `find`

If you know the exact name of the file you're looking for:
```bash
find -type f -name "app.txt"
```
This will give us the exact location of all files with the specified name (within the current folder).  

To find all .txt files inside the DevOps folder:
```bash
find ./DevOps -type f -name "*.txt"
```

## `find` + pipe

To know how many .txt files there is within that same folder: 
```bash
find ./DevOps -type f -name "*.txt" | grep wc -l
```

##

---
7/32
