# Unix Tips, Tools & Bash Scripting

## Tips
### Brace expansion

```
$ tree -d
.
├── 2019
│   ├── 01
│   ├── 02
│   ├── 03
│   ├── 04
│   ├── 05
│   ├── 06
│   ├── 07
│   ├── 08
│   ├── 09
│   ├── 10
│   ├── 11
│   └── 12
└── 2020
    ├── 01
    ├── 02
    ├── 03
    ├── 04
    ├── 05
    ├── 06
    ├── 07
    ├── 08
    ├── 09
    ├── 10
    ├── 11
    └── 12
```

```
# Supplied integers may be prefixed with `0` to force each term to have the same width
mkdir -p 20{19,20}/{01..12}
```

```
cp file{,.bak}
```

### Pushd, popd
- The `pushd` command saves the current working directory in memory so it can be returned to at any time.

- The `popd` command returns to the path at the top of the directory stack. 

- The directory stack is accessed by the command `dirs` in Unix.

### Find all hard links to a given file

```
find . -samefile /path/to/file
```

### Process Substitution `<()`

```
# Process substitution uses /dev/fd/<n> files 
# to send the results of the process(es) 
# within parentheses to another process.

$ echo <(true)
/dev/fd/63

$ diff <(ls -la) <(ls -lat)

$ grep -o '<div' <(curl -sL www.in.gr) | wc -l
```

## Unix Tools
1. Grep
2. Awk
3. Find

### 1. Grep
#### Basic Syntax
```
grep [OPTIONS] PATTERN [FILE...]
```

#### Match word

```
# The expression is searched for as a word 
# (as if surrounded by `[[:<:]]' and `[[:>:]]'; see re_format(7)).

grep -w "pattern" filename
grep '[[:<:]]pattern[[:>:]]' filename
```

```
$ grep test filename
$ grep -w test filename
```

#### Display extra lines (-A, -B, -C)

**After:**
`grep -A NUM "pattern" filename`

**Before:**
`grep -B NUM "pattern" filename`

**Around:**
`grep -C NUM "pattern" filename`

```
$ grep -A 1 line5 filename
$ grep -B 1 line5 filename
$ grep -C 1 line5 filename
```

#### Count the number of matches
```
grep -c "pattern" filename
```

```
$ grep -c test filename
$ grep -wc test filename
```

####  Show only the matched string
```
grep -o "pattern" filename
```

```
$ grep -o test filename
```

#### Stop reading a file after NUM matching lines
```
grep -m NUM "pattern" filename
```

```
$ grep -m 1 test filename
```

#### Quiet;  do  not  write anything to standard output
```
# Search until a match has been found (less expensive)
grep -q "pattern" filename
```

```
$ grep -q test filename
$ echo $?
```

#### End of command options
```
$ grep -- -v filename
```

### 2. Awk
AWK is a programming language designed for text processing and typically used as a data extraction and reporting tool. It is a standard feature of most Unix-like operating systems.

AWK is automatically sets some useful variables, like:

- **NF**  number of fields in the current line
- **NR** current line number
- **$0** current line
- **$1, $2, $3, ...** value of each field 
- more

#### Basic Syntax
```
awk 'BEGIN {start_action} {action} END {stop_action}' filename
```
- The actions in the begin block are performed before processing the file 
- The actions in the end block are performed after processing the file
- The rest of the actions are performed while processing the file

#### Examples
```
awk '{ count++ } END { print count }' filename
```

```
awk '{ count += length($0) } END { print count }' filename
```

```
awk '{ print NR, $0 }' filename
```

```
echo 'hello123world' | awk -F'[0-9]+' '{ print $2 }'
```

```
ls -la | awk '{ print $3, $4 }'
```


```
ls -la | awk '{ print $3, $4, $NF }'
```

```
awk '{ hosts[$1]++ } END { for (i in hosts) print i, hosts[i] }' access.log \
  | sort -nr -k2 \
  | head -10     \
  | column -t
```

```
awk '{ if ($(NF-1) >= 400) hosts[$1]++ } END { for (i in hosts) print i, hosts[i] }' access.log \
  | sort -nr -k2 \
  | head -10     \
  | column -t
```

```
awk '{ hosts[$1] += $NF } END { for (i in hosts) printf("%s %d KB\n", i, hosts[i]/2^10) }' access.log \
  | sort -nr -k2 \
  | head -10     \
  | column -t
```

### 3. Find
```
find . -type f -exec wc -l {} \;
echo $?

find . -type f | xargs wc -l
echo $?
```

```
find . -type f -exec wc -l {} \+
find . -type f -print0 | xargs -0 wc -l
find . -type f -print0 | xargs -0 -n2 wc -l
```

```
find . -name '*.php' -type f -exec php -l {} \;
find . -name '*.php' -type f -print0 | xargs -0 php -l
find . -name '*.php' -type f -print0 | xargs -0 -n1 php -l
```

#### Exit code
`find -exec` returns the exit code of find itself instead of the subcommand

`find | xargs` returns the exit code of the subcommand

#### Should I use -exec or xargs?
If you need to stop the execution on failed subcommands, you need to use xargs.

If you want or need to continue the on failed subcommands you have to use find -exec.

## Bash shell scripting

1. `[` and `[[` test commands
2. Command substitution
3. Loops
4. How to read files
5. Exit status
6. Functions
7. Useful Options for Scripting
8. Shell script analysis

### 1. `[` and `[[` test commands

```
$ [ $v == "" ]
bash: [: ==: unary operator expected
$ echo $?
2
```

```
$ [ x$v == "x" ]
$ echo $?
0
```

```
$ [ "$v" == "" ]
$ echo $?
0
```

```
# Double [[ ]] are an extension to the standard []
# They support some extra operations, like =~ for regex matching

$ [[ $v == "" ]]
$ echo $?
0
```

**`[[` doesn't like the `-a` (and) and `-o` (or) operators**

```
$ [[ "$v" == "" -a "$z" == "" ]]
bash: syntax error in conditional expression
bash: syntax error near `-a'
```

**Solution**

```
$ [ "$v" == "" ] && [ "$z" == "" ]

# OR

[[ "$v" == "" ]] && [[ "$z" == "" ]]
```

### 2. Command substitution
Command substitution allows the output of a command to replace the command itself. Command substitution occurs when a command is enclosed as follows:

```
$(command)
```

```
`command`
```

Bash performs the expansion by executing command in a subshell environment and replacing the command substitution with the standard output of the command, with any trailing newlines deleted. 

Embedded newlines are not deleted, but they may be removed during word splitting. 

The command substitution `$(cat file)` can be replaced by the equivalent but faster `$(< file)`. 

```
#!/bin/bash

dt=$(date -u +%d.%m.%Y\ %H:%M)
echo "$dt"
```

### 3. Loops

```
#!/bin/bash

for x in 1 2 3; do
    echo "$x"
done
```

```
#!/bin/bash

for x in {1..3}; do
    echo "$x"
done
```

```
#!/bin/bash

for file in *; do
    echo "$file"
done
```

```
#!/bin/bash

# Looping over find's output bad practice.
# Filenames can contain any character,
# so, there is no printable character
# you can reliably use to delimit filenames.
#
# A null byte is the only correct way to delimit filenames:
# e.g. `find -print0` + `xargs -0`
for file in $(ls); do
    echo "file:$file"
done
```

```
# Add IFS= so that read won't trim leading and trailing whitespace from each line
# Add -r to read to prevent from backslashes from being interpreted as escape sequences
# Use printf in place of echo which is safer if $line is a string like -n which echo would interpret as a flag

#!/bin/bash

find . -maxdepth 1 -print0 | xargs -0 -n1 |
while IFS= read -r line; do 
	printf 'line:%s\n' "$line";
done
```

```
#!/bin/bash

while IFS= read -r line; do 
	printf 'line:%s\n' "$line";
done < <(find . -maxdepth 1 -print0 | xargs -0 -n1)
```

### 4. How to read files

**Problem**

```
    # in.txt
*
```

```
#!/bin/bash

for line in $(cat in.txt); do
    echo "line:$line"
done
```

**Let's quote the command and try again:**

```
#!/bin/bash

for line in "$(cat in.txt)"; do
    echo "line:$line"
done
```

**`read` to the rescue!**

```
while read -r line; do
    echo "line:$line"
done < in.txt
```

```
#!/bin/bash

# Add IFS= so that read won't trim leading and trailing whitespace from each line
# Add -r to read to prevent from backslashes from being interpreted as escape sequences
# Use printf in place of echo which is safer if $line is a string like -n which echo would interpret as a flag

file=${1:-/dev/stdin}
while IFS= read -r line; do
    printf '%s\n' "$line"
done < "$file" # data from $file is redirected to stdin (fd0)
```

```
#!/bin/bash

while IFS=: read -r username password userid groupid comment homedir cmdshell; do
	echo "$username, $userid, $comment $homedir"
done < /etc/passwd
```

### 5. Exit status
- 8bit (0-255)
- `0` success
- `>0` failure
- `$?` code
- `exit 0` is this necessary at the end of the script?

### 6. Functions
**Definition**

```
function myfunc {
}
```

**OR**

```
# POSIX compliant
myfunc() { 
}
```

**Arguments**

- `$0` Script name
- `$1` 1st argument
- `$2` 2nd argument
- `$n` Individual arguments on the command line (positional parameters). The
Bourne shell allows only nine parameters to be referenced directly (n=1–9); Bash allows n to be greater than 9 if specified as `${n}`
- `$#` Number of arguments
- `$@, $*` All arguments on the command line ($1 $2)
- `"$@"` All arguments on the command line, individually quoted ("$1" "$2"...)
- `"$*"` All arguments on the command line as one string ("$1 $2..."). The values are separated by the first character in $IFS (space if unset)

**Invocation**

```
function myfunc {
	echo "$1 $2"
}

myfunc hello world
```

**Variables scope**

- By default all variables are global
- Modifying a variable in a function changes it in the whole script
- Use `local` to restrict their scope

```
function myfunc {
	# x is only visible to this fucntion
	local x="test"
}
```
**Pass arguments by reference**

```
#!/bin/bash

function myfunc {
	# -n needs bash 4.3+
   local -n ref=$1
   
   ref="changed"
}

v="init"
myfunc v
echo "$v"
```

**Returning values**

The return command causes a function to exit with the return value (*exit status*) specified by N (*integer*, 0-255) and the syntax is:

- `return N`
- If N is not specified, the return status is that of the last command
- The `return` command terminates the function
- The `return` command is not necessary when the return value is that of the last command executed

**Examples**

```
#!/bin/bash

function is_root_user {
    [ $(id -u) -eq 0 ]
}

if is_root_user; then
    echo "root user"
else
    echo "normal user"
fi
```

```
#!/bin/bash

# set -e will cause the entire script to exit
# if any function returns a value != 0
set -e

function func0 {
    return 0
}

function func1 {
    return 1
}

function func2 {
    return 2
}

func0
func1
func2
```

**Returning a string or word from a function**

- You cannot return a word or anything else from a function
- However, you can use echo or printf command to send back output easily to the script

```
#!/bin/bash

function myfunc {
	echo "result"
}

x=$(myfunc)

```

**Calling exit**

```
#!/bin/bash

function myfunc {
	exit 2
}

myfunc
echo "hello"
```

```
#!/bin/bash

function myfunc {
    exit 2
}

( myfunc )
echo "hello"
```

**List functions**

Function name + body:

```declare -f```

Only function name:

```declare -F```

**Overriding builtin functions**

```
$ function cd {
	ls -la $1
}

$ command -V cd
cd is a function

$ cd /tmp
$ builtin cd /tmp
$ unset -f cd
$ cd /tmp
```

### 7. Useful Options for Scripting

```
set -o errexit # OR set -e
set -o xtrace  # OR set -x
```

- The errexit option tells bash to exit the script if any command fails.
- The xtrace option outputs each command as it is being run. This is really useful for seeing what command was actually run if (for example) you are using variables within your commands. It also helps you see the order in which commands are being run.

### 8. Shell script analysis
```
# Download it from https://www.shellcheck.net/
shellcheck script.sh
```

## Links / Sources
- https://devhints.io/bash
- https://www.gnu.org/software/bash/manual/
- http://wiki.bash-hackers.org/
- http://www.tldp.org/LDP/abs/html/
- http://mywiki.wooledge.org/BashFAQ/
- https://www.everythingcli.org/find-exec-vs-find-xargs/
- https://github.com/learnbyexample/Command-line-text-processing
- https://bash.cyberciti.biz/guide/
- [Bash Pocket Reference, 2nd Edition](https://www.oreilly.com/library/view/bash-pocket-reference/9781491941584/)
- [Learn Bash the Hard Way](https://leanpub.com/learnbashthehardway)
- man bash
