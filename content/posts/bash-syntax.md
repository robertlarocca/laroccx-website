---
author: "Robert LaRocca"
title: "Bash Syntax Guide"
date: 2021-07-08T03:45:00-04:00
categories:
  - development
tags:
  - development
  - scripting
  - bash
  - shell
draft: false
---

This post is more or less a Bash syntax reference guide and isn't intended to be detailed user guide for learning how to Bash script. It does however, include most of the syntax you'll need to create shell scripts.

Much of this content is borrowed from other authors and `open-source` content creators. I've simply combined their works into a easy to follow format. I'll do my best to credit the original author and provide sources whenever possible.

<!--more-->

## Example Script

```bash
#!/usr/bin/env bash

NAME="Robert"
echo "Hello $NAME!"
```

## Variables

```bash
NAME="Robert"
echo $NAME        # Robert
echo "$NAME"      # Robert
echo "${NAME}!"   # Robert!
```

## String Quotes

```bash
NAME="Robert"
echo "Hello $NAME"    # Hello Robert
echo 'Hello $NAME'    # Hello $NAME
```

## Command Substitution

```bash
echo "The current directory is $(pwd)"
```

See [command substitution](http://wiki.bash-hackers.org/syntax/expansion/cmdsubst) for more information.

## Conditional Execution

```bash
git commit && git push
git commit || echo "Commit Failed!"
```

## Functions

```bash
display_name() {
  NAME="Robert"
  echo "$NAME"
}

echo "Hello, My name is $(display_name)"
```

## Conditionals

```bash
if [ -z "$variable" ]; then
  echo "Variable is empty"
elif [ -n "$variable" ]; then
  echo "Variable is not empty"
fi
```

## Strict Mode

```bash
set -euo pipefail
IFS=$'\n\t'
```

See [unofficial bash strict mode](http://redsymbol.net/articles/unofficial-bash-strict-mode/) for more information.

## Brace Expansion

```bash
echo {A,B}.sh
```

| Expression | Description         |
| ---------- | ------------------- |
| `{A,B}`    | Same as `A B`       |
| `{A,B}.sh` | Same as `A.sh B.sh` |
| `{1..5}`   | Same as `1 2 3 4 5` |

See [brace expansion](http://wiki.bash-hackers.org/syntax/expansion/brace) for more information.

## Parameter Expansions

### Basics

```bash
name="Robert"
echo ${name}          # "Robert"
echo ${name/R/r}      # "robert" (substitution)
echo ${name:0:2}      # "Ro" (slicing)
echo ${name::2}       # "Ro" (slicing)
echo ${name::-1}      # "Rob" (slicing)
echo ${name:(-1)}     # "t" (slicing from right)
echo ${name:(-2):1}   # "r" (slicing from right)
```

```bash
length=2
echo ${name:0:length}   # "Ro"
```

See [parameter expansion](http://wiki.bash-hackers.org/syntax/pe) for more information.

```bash
STR="/path/to/foo.cpp"
echo ${STR%.cpp}      # /path/to/foo
echo ${STR%.cpp}.o    # /path/to/foo.o
echo ${STR%/*}        # /path/to

echo ${STR##*.}       # cpp (extension)
echo ${STR##*/}       # foo.cpp (basepath)

echo ${STR#*/}        # path/to/foo.cpp
echo ${STR##*/}       # foo.cpp

echo ${STR/foo/bar}   # /path/to/bar.cpp
```

```bash
STR="Hello World!"
echo ${STR:6:5}     # "World"
echo ${STR: -5:5}   # "orld!"
```

```bash
SRC="/path/to/foo.cpp"
BASE=${SRC##*/}     # "foo.cpp" (basepath)
DIR=${SRC%$BASE}    # "/path/to/" (dirpath)
```

## Substitution

| Code              | Description         |
| ----------------- | ------------------- |
| `${FOO%suffix}`   | Remove suffix       |
| `${FOO#prefix}`   | Remove prefix       |
| `${FOO%%suffix}`  | Remove long suffix  |
| `${FOO##prefix}`  | Remove long prefix  |
| `${FOO/from/to}`  | Replace first match |
| `${FOO//from/to}` | Replace all         |
| `${FOO/%from/to}` | Replace suffix      |
| `${FOO/#from/to}` | Replace prefix      |

## Comments

```bash
# This is a single line comment!
```

```bash
# This is a
# multi-line
# comment
```

```bash
: '
This is a
multi-line
comment
'
```

## Substrings

| Expression      | Description                    |
| --------------- | ------------------------------ |
| `${FOO:0:3}`    | Substring _(position, length)_ |
| `${FOO:(-3):3}` | Substring from the right       |

## Length

| Expression | Description      |
| ---------- | ---------------- |
| `${#FOO}`  | Length of `$FOO` |

## Manipulation

```bash
STR="HELLO WORLD!"
echo ${STR,}    # hELLO WORLD! (first letter lowercase)
echo ${STR,,}   # hello world! (all lowercase)

STR="hello world!"
echo ${STR^}    # Hello world! (first letter uppercase)
echo ${STR^^}   # HELLO WORLD! (all uppercase)
```

## Default Values

| Expression        | Description                                              |
| ----------------- | -------------------------------------------------------- |
| `${FOO:-val}`     | `$FOO`, or `val` if unset (or null)                      |
| `${FOO:=val}`     | Set `$FOO` to `val` if unset (or null)                   |
| `${FOO:+val}`     | `val` if `$FOO` is set (and not null)                    |
| `${FOO:?message}` | Show error message and exit if `$FOO` is unset (or null) |

Omitting the `:` removes the (non)nullity checks. `${FOO-val}` expands to `val` if unset otherwise `$FOO`.

## Loops

### Basic For Loop

```bash
for i in /etc/rc.*; do
  echo $i
done
```

### C-style For Loop

```bash
for ((i = 0 ; i < 100 ; i++)); do
  echo $i
done
```

### Ranges

```bash
for i in {1..5}; do
  echo "Welcome $i"
done
```

### Step Size

```bash
for i in {5..50..5}; do
  echo "Welcome $i"
done
```

### Reading Lines

```bash
cat file.txt | while read line; do
  echo $line
done
```

### Forever

Be careful when using infinite loops! If, not used properly may crash a running system.

```bash
while true; do
  ···
done
```

## Functions

### Defining functions

```bash
myfunc() {
  echo "Hello $1"
}
```

This function is the same as above, but with alternate syntax.

```bash
function myfunc() {
  echo "Hello $1"
}
```

```bash
myfunc "Robert"
```

### Returning Values

```bash
myfunc() {
  local myresult='some value'
  echo $myresult
}
```

```bash
result="$(myfunc)"
```

### Raising Errors

```bash
myfunc() {
  return 1
}
```

```bash
if myfunc; then
  echo "success"
else
  echo "failure"
fi
```

### Arguments

| Expression | Description                           |
| ---------- | ------------------------------------- |
| `$#`       | Number of arguments                   |
| `$*`       | All arguments                         |
| `$@`       | All arguments, starting from first    |
| `$1`       | First argument                        |
| `$_`       | Last argument of the previous command |

See [special parameters](http://wiki.bash-hackers.org/syntax/shellvars#special_parameters_and_shell_variables) for more information.

## Conditionals

### Conditions

Using `[[` is actually a command-line program that returns either `0` (true) or `1` (false). Any program that obeys the same logic can be used as conditionals. All base utilities, such as `grep` or `ping` support conditionals.

| Condition                | Description           |
| ------------------------ | --------------------- |
| `[[ -z STRING ]]`        | Empty string          |
| `[[ -n STRING ]]`        | Not empty string      |
| `[[ STRING == STRING ]]` | Equal                 |
| `[[ STRING != STRING ]]` | Not Equal             |
| `[[ NUM -eq NUM ]]`      | Equal                 |
| `[[ NUM -ne NUM ]]`      | Not equal             |
| `[[ NUM -lt NUM ]]`      | Less than             |
| `[[ NUM -le NUM ]]`      | Less than or equal    |
| `[[ NUM -gt NUM ]]`      | Greater than          |
| `[[ NUM -ge NUM ]]`      | Greater than or equal |
| `[[ STRING =~ STRING ]]` | Regexp                |
| `(( NUM < NUM ))`        | Numeric conditions    |

#### More Conditions

| Condition            | Description              |
| -------------------- | ------------------------ |
| `[[ -o noclobber ]]` | If OPTIONNAME is enabled |
| `[[ ! EXPR ]]`       | Not                      |
| `[[ X && Y ]]`       | And                      |
| `[[ X \|\| Y ]]`     | Or                       |

### File Conditions

| Condition               | Description             |
| ----------------------- | ----------------------- |
| `[[ -e FILE ]]`         | Exists                  |
| `[[ -r FILE ]]`         | Readable                |
| `[[ -h FILE ]]`         | Symbolic Link           |
| `[[ -d FILE ]]`         | Directory               |
| `[[ -w FILE ]]`         | Writable                |
| `[[ -s FILE ]]`         | Size `>` 0 bytes        |
| `[[ -f FILE ]]`         | File                    |
| `[[ -x FILE ]]`         | Executable              |
| `[[ FILE1 -nt FILE2 ]]` | 1 is more recent than 2 |
| `[[ FILE1 -ot FILE2 ]]` | 2 is more recent than 1 |
| `[[ FILE1 -ef FILE2 ]]` | Same files              |

### Examples

```bash
# String
if [[ -z "$string" ]]; then
  echo "String is empty"
elif [[ -n "$string" ]]; then
  echo "String is not empty"
else
  echo "This never happens"
fi
```

```bash
# Combinations
if [[ X && Y ]]; then
  ...
fi
```

```bash
# Equal
if [[ "$A" == "$B" ]]
```

```bash
# Regex
if [[ "A" =~ . ]]
```

```bash
if (( $a < $b )); then
  echo "$a is smaller than $b"
fi
```

```bash
if [[ -e "file.txt" ]]; then
  echo "file exists"
fi
```

## Arrays

### Defining Arrays

```bash
Fruits=('Apple' 'Banana' 'Orange')
```

```bash
Fruits[0]="Apple"
Fruits[1]="Banana"
Fruits[2]="Orange"
```

### Working with Arrays

```bash
echo ${Fruits[0]}       # Element #0
echo ${Fruits[-1]}      # Last element
echo ${Fruits[@]}       # All elements, space-separated
echo ${#Fruits[@]}      # Number of elements
echo ${#Fruits}         # String length of the 1st element
echo ${#Fruits[3]}      # String length of the Nth element
echo ${Fruits[@]:3:2}   # Range (from position 3, length 2)
echo ${!Fruits[@]}      # Keys of all elements, space-separated
```

### Operations

```bash
Fruits=("${Fruits[@]}" "Watermelon")      # Push
Fruits+=('Watermelon')                    # Also Push
Fruits=( ${Fruits[@]/Ap*/} )              # Remove by regex match
unset Fruits[2]                           # Remove one item
Fruits=("${Fruits[@]}")                   # Duplicate
Fruits=("${Fruits[@]}" "${Veggies[@]}")   # Concatenate
lines=(`cat "logfile"`)                   # Read from file
```

### Iteration

```bash
for i in "${arrayName[@]}"; do
  echo $i
done
```

## Dictionaries

### Defining

```bash
declare -A sounds
```

```bash
sounds[dog]="bark"
sounds[cow]="moo"
sounds[bird]="tweet"
sounds[wolf]="howl"
```

Declares `sound` as a dictionary object (associative array).

### Working with Dictionaries

```bash
echo ${sounds[dog]}   # Dog's sound
echo ${sounds[@]}     # All values
echo ${!sounds[@]}    # All keys
echo ${#sounds[@]}    # Number of elements
unset sounds[dog]     # Delete dog
```

### Iteration

#### Iterate over Values

```bash
for val in "${sounds[@]}"; do
  echo $val
done
```

#### Iterate over Keys

```bash
for key in "${!sounds[@]}"; do
  echo $key
done
```

## Options

### Basic Options

```bash
set -o noclobber    # Avoid overlay files (echo "hi" > foo)
set -o errexit      # Used to exit upon error, avoiding cascading errors
set -o pipefail     # Unveils hidden failures
set -o nounset      # Exposes unset variables
```

### Glob Options

```bash
shopt -s nullglob     # Non-matching globs are removed  ('*.foo' => '')
shopt -s failglob     # Non-matching globs throw errors
shopt -s nocaseglob   # Case insensitive globs
shopt -s dotglob      # Wildcards match dotfiles ("*.sh" => ".foo.sh")
shopt -s globstar     # Allow ** for recursive matches ('lib/**/*.rb' => 'lib/a/b/c.rb')
```

Set `GLOBIGNORE` as a colon-separated list of patterns to be removed from glob matches.

## History

### Commands

| Command               | Description                               |
| --------------------- | ----------------------------------------- |
| `history`             | Show history                              |
| `shopt -s histverify` | Don't execute expanded result immediately |

### Expansions

| Expression   | Description                                          |
| ------------ | ---------------------------------------------------- |
| `!$`         | Expand last parameter of most recent command         |
| `!*`         | Expand all parameters of most recent command         |
| `!-n`        | Expand `n`th most recent command                     |
| `!n`         | Expand `n`th command in history                      |
| `!<command>` | Expand most recent invocation of command `<command>` |

### Operations

| Code                 | Description                                                           |
| -------------------- | --------------------------------------------------------------------- |
| `!!`                 | Execute last command again                                            |
| `!!:s/<FROM>/<TO>/`  | Replace first occurrence of `<FROM>` to `<TO>` in most recent command |
| `!!:gs/<FROM>/<TO>/` | Replace all occurrences of `<FROM>` to `<TO>` in most recent command  |
| `!$:t`               | Expand only basename from last parameter of most recent command       |
| `!$:h`               | Expand only directory from last parameter of most recent command      |

`!!` and `!$` can be replaced with any valid expansion.

### Slices

| Code     | Description                                                                              |
| -------- | ---------------------------------------------------------------------------------------- |
| `!!:n`   | Expand only `n`th token from most recent command (command is `0`; first argument is `1`) |
| `!^`     | Expand first argument from most recent command                                           |
| `!$`     | Expand last token from most recent command                                               |
| `!!:n-m` | Expand range of tokens from most recent command                                          |
| `!!:n-$` | Expand `n`th token to last from most recent command                                      |

`!!` can be replaced with any valid expansion i.e. `!cat`, `!-2`, `!42`, etc.

## Miscellaneous

### Numeric Calculations

```bash
$((a + 200))    # Add 200 to $a
```

```bash
$(($RANDOM%200))    # Random number 0..199
```

### Sub-Shells

```bash
(cd /var/tmp; echo "I'm now in $PWD")
pwd   # still in the original directory
```

### Redirection

```bash
python hello.py > output.txt    # stdout to (file)
python hello.py >> output.txt   # stdout to (file), append
python hello.py 2> error.log    # stderr to (file)
python hello.py 2>&1            # stderr to stdout
python hello.py 2>/dev/null     # stderr to (null)
python hello.py &>/dev/null     # stdout and stderr to (null)
```

```bash
python hello.py < foo.txt   # feed foo.txt to stdin for python
```

### Inspecting Commands

```bash
command -V cd   # "cd is a function/alias/whatever"
```

### Trap Errors

```bash
trap 'echo Error at about $LINENO' ERR
```

or

```bash
traperr() {
  echo "ERROR: ${BASH_SOURCE[1]} at about ${BASH_LINENO[0]}"
}

set -o errtrace
trap traperr ERR
```

### Case and Switch

```bash
case "$1" in
  start | up)
    vagrant up
    ;;
  *)
    echo "Usage: $0 {start|stop|ssh}"
    ;;
esac
```

### Source Relative

```bash
source "${0%/*}/../share/foo.sh"
```

### Format and Print Data

Use the `printf` command to format and print data.

```bash
printf "Hello %s, I'm %s" Sven Olga     # "Hello Sven, I'm Olga
printf "1 + 1 = %d" 2                   # "1 + 1 = 2"
printf "How you print a float: %f" 2    # "This is how you print a float: 2.000000"
```

### Directory of Script

```bash
DIR="${0%/*}"
```

### Getting Options

```bash
while [[ "$1" =~ ^- && ! "$1" == "--" ]]; do case $1 in
  -V | --version )
    echo $version
    exit
    ;;
  -s | --string )
    shift; string=$1
    ;;
  -f | --flag )
    flag=1
    ;;
esac; shift; done
if [[ "$1" == '--' ]]; then
  shift
fi
```

### Heredoc

```sh
cat <<EOF_XYZ
hello world
EOF_XYZ
```

```sh
cat <<-EOF_XYZ
  hello world
  EOF_XYZ
```

### Reading User Input

```bash
echo -n "Proceed? [y/n]: "
read ans
echo $ans
```

```bash
read -n 1 ans   # Just one character
```

### Special Variables

| Expression | Description                  |
| ---------- | ---------------------------- |
| `$?`       | Exit status of last task     |
| `$!`       | PID of last background task  |
| `$$`       | PID of shell                 |
| `$0`       | Filename of the shell script |

See [special parameters](http://wiki.bash-hackers.org/syntax/shellvars#special_parameters_and_shell_variables) for more information.

### Goto Previous Directory

```bash
pwd   # /home/user/foo
cd bar/
pwd   # /home/user/foo/bar
cd -
pwd   # /home/user/foo
```

### Check for Command Result

```bash
if ping -c 1 www.laroccx.com; then
  echo "You're connected to the internet!"
else
  echo "You're not connected to the internet!"
fi
```

## Sources

- [The Bash Hackers Wiki](http://wiki.bash-hackers.org/)
- [Special Parameters and Shell Variables](http://wiki.bash-hackers.org/syntax/shellvars)
- [ShellCheck A Shell Script Analysis Tool](https://www.shellcheck.net/)
