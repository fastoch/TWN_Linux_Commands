# Resources

- https://www.youtube.com/watch?v=fwP2JW_VnZI
- https://gitlab.com/twn-youtube/twn-linux-commands
- https://linuxopsys.com/find-exec-command-in-linux-with-examples

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

## `find` + pipe + `xargs` + `cat` + `grep`

Now, the idea is to find all .txt files within a given folder and to search through the contents of these files for specific error messages.  

```bash
find ./logs/2026-02 -type f -name "*.txt" | xargs cat 
```

- `cat` displays the contents of a file
- `xargs` takes input from standard input (stdin) and turns it into arguments for another command

The above command returns the contents of all .txt files within the specified folder.  
Now we want to filter these contents for specific error messages: 
```bash
find ./logs/2026-02 -type f -name "*.txt" | xargs cat | grep "ERROR"
```

# Let's add `sort` and `uniq` to the command pipeline

Then, we can sort the results alphabetically: 
```bash
find ./logs/2026-02 -type f -name "*.txt" | xargs cat | grep "ERROR" | sort
```
The Unix/Linux `sort` command sorts entire lines by default.  
It treats each full line as the sort key, using alphabetical order based on the first differing character.  

But we can sort the results based on the 4th whitespace-delimited field (4th column): 
```bash
find ./logs/2026-02 -type f -name "*.txt" | xargs cat | grep "ERROR" | sort -k4
```

And finally, we can remove duplicates via the `uniq` command: 
```bash
find ./logs/2026-02 -type f -name "*.txt" | xargs cat | grep "ERROR" | sort -k4 | uniq
```

>[!important]
>Since `uniq` only detects adjacent duplicated lines, it is most often used in a commande pipeline immediately after the `sort` command.

If we don't care about the uniqueness of the first two fields (could be timestamps):
```bash
find ./logs/2026-02 -type f -name "*.txt" | xargs cat | grep "ERROR" | sort -k4 | uniq -f3
```

## Saving the output to a file with redirection

- Pipe `|` sends output of a command to another command
- Redirection `>` sends output to a file, it overwrites any existing contents in this file

```bash
find ./logs/2026-02 -type f -name "*.txt" | xargs cat | grep "ERROR" | sort -k4 | uniq -f3 > errors.log
```
If the errors.log file does not exist in the current directory, it will be created.  
If it already exists, it will be overwritten.  

- `>>` appends the command output at the end of the file, so that existing contents don't get overwritten

```bash
find ./logs/2026-02 -type f -name "*.txt" | xargs cat | grep "ERROR" | sort -k4 | uniq -f3 >> errors.log
```

## Making backups before deleting old log files

Let's say we have an automated script running periodically that deletes logs older than 3 months.  
Those logs may still contain valuable information, so we should back them up somewhere before deletion.  

- `-exec` is a parameter of the `find` command which allows us to execute a specific command on every found file
```bash
find ./logs/2026-02 -type f -name "*.txt" -exec cp {} /Web_app/logs/2026-02-backup + 
```
For every .txt file inside the specifed folder, execute the copy command.  

- in a `find ... -exec` command, curly brackets `{}` act as a placeholder that is replaced by the name of each found file
- The plus sign `+` terminates the `-exec` action, telling `find` where the command argument ends
  - This `+` delimiter works with `-exec` to pass all found files as arguments to the given command at once
- Note that we could also use `\;` instead of `+` as a delimiter for the `-exec` command argument

More details about `find ... -exec` commands: https://linuxopsys.com/find-exec-command-in-linux-with-examples  

## Using `rsync`

The previous command can be improved.  

All the files we're targeting are being copied to the same backup folder, and since a folder cannot contain two or more files 
with the same name, what will happen if our `find` command returns multiple files with the exact same name?  
> Only the last copied file will be backed up, previous files having the same name will be overwritten.

That's because `cp` performs a straightforward copy of each found file to the destination folder, replacing any existing target file.  

To back up our files without risking losing data in the process, we need to use `rsync -R` instead of `cp`.  
The `-R` (`--relative`) option for `rsync` tells it to preserve the relative path structure of each source file when copying it to the destination.  

Our final command:
```bash
find ./logs/2026-02 -type f -name "*.txt" -exec rsync -R {} /Web_app/logs/2026-02-backup + 
```

---
Tutorial completed
