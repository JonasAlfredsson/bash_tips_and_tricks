# bash_tips_and_tricks

I don't write Bash code that often, so when I need to do it I usually have to
Google the same stuff every time because I cannot remember exactly how it should
be done. So in order to reduce the time wasted on remembering what search term
I used the last time I found the best guide I will just collect them here (with
links back to the source of course).

I think I will keep everything in a single document so it will be easier to
`Ctrl+F` stuff.


## Settings
Bash scripts has a few [settings][14] which might be of interest to include at
the top like this:

```bash
#!/bin/bash
set -eou pipefail
```

- `#!/bin/bash`: A "shebang" - Allows the OS to know that Bash should be used
                 when executing a file directly like `./script.sh`.
- `set -e`: Have the script exit if any command fails.
- `set -u`: Have the script fail if any variable is unset.
- `set -o`: Provide extra options, in this case "pipefail" -> Have the script
            exit if any command in a pipe fails.


## Variables

Variables is something you always have to work with and you often have to
set them and/or determine if they are empty, so here are a couple of different
ways to handle that. The [Bash Hackers Wiki][1] has quite a lot more info if
you want to dive deeper.

But some common definitions that you need to know going forward:

- **set** means `VARIABLE` is non-empty (`VARIABLE="something"`)
- **empty** means `VARIABLE` is empty/null (`VARIABLE=""`)
- **unset** means `VARIABLE` does not exist (`unset VARIABLE`)

### Use Default Value
When reading a variable you sometimes want to either use a default value or
return an error in case it is unset. Here is a table with what happens in the
different cases ([source][3]).

| Expression in script: | When `FOO="world"` | When `FOO=""` | `unset FOO`   |
|-----------------------|--------------------|---------------|---------------|
| `echo ${FOO:-hello}`  | `world`            | `hello`       | `hello`       |
| `echo ${FOO-hello}`   | `world`            | `""`          | `hello`       |
| `echo ${FOO:=hello}`  | `world`            | `FOO=hello`   | `FOO=hello`   |
| `echo ${FOO=hello}`   | `world`            | `""`          | `FOO=hello`   |
| `echo ${FOO:?hello}`  | `world`            | `error, exit` | `error, exit` |
| `echo ${FOO?hello}`   | `world`            | `""`          | `error, exit` |
| `echo ${FOO:+hello}`  | `hello`            | `""`          | `""`          |
| `echo ${FOO+hello}`   | `hello`            | `hello`       | `""`          |

What is important to understand here is that the `FOO` variable will retain its
original `world` value in all but the three cases where `FOO=hello` is set as
the result. In those cases `FOO` will first be changed to this new value before
the result is returned and `echo`:ed in this case.

So to create a "use environment variable if set, else use default" definition
it would looks something like this

```bash
: ${FOO:=hello}
```

where the leading colon is important so the variable is not accidentally
executed when read.

### Empty Check
A simple `if` check to see if a variable is empty or not ([source][2]).

```bash
if [ -z "${VAR}" ]; then
    echo "VAR is unset or set to the empty string"
fi
if [ -z "${VAR+set}" ]; then
    echo "VAR is unset"
fi
if [ -z "${VAR-unset}" ]; then
    echo "VAR is set to the empty string"
fi
if [ -n "${VAR}" ]; then
    echo "VAR is set to a non-empty string"
fi
if [ -n "${VAR+set}" ]; then
    echo "VAR is set, possibly to the empty string"
fi
if [ -n "${VAR-unset}" ]; then
    echo "VAR is either unset or set to a non-empty string"
fi
```

## Strings
String manipulation is a bit of continuation of the Variables section above,
but is different enough that it gets its own [section][15].

### Casing
[Normalize casing][16] might sometimes be necessary to simplify comparisons.

```bash
string="to uppercase"
echo "${string^}"  # To uppercase
echo "${string^^}" # TO UPPERCASE
```

```bash
string="TO LOWERCASE"
echo "${string,}"  # tO LOWERCASE
echo "${string,,}" # to lowercase
```

### Equality
Simple check to see if the strings are equal.

```bash
if [[ "${s1}" == "${s2}" ]]; then
    echo "Equal"
fi
```

### Substring Checking
Here we look for a substring in a longer sentence.

```bash
full_sentence="This is a long sentence"
part="is a long"
```

```bash
if [[ "${full_sentence}" == *"${part}"* ]]; then
    echo "Substring found!"
fi
```

```bash
# The "-i" makes grep case insensitive.
if echo "${full_sentence}" | grep -i -q "${part}"; then
    echo "Substring found!"
fi
```

### Regex Matching
A more powerful search with the help of regex.

```bash
if [[ "${sentence}" =~ ^(one|two) ]]; then
   echo "Sentence starts with either 'one' or 'two'."
fi
```

```bash
if echo "${sentence}" | grep -E -q '^(one|two)'; then
    echo "Sentence starts with either 'one' or 'two'."
fi
```

### Multiline Concatenation
Instead of storing a lot of data in temporary files it is possible to work with
multiline variables in memory instead.

```bash
multiline_variable="This is a
multiline variable
where we will
"
multiline_variable+="add another line not ending in newline"
multiline_variable+=$'\n' # Add missing newline.
multiline_variable+="$(cat "${filepath}")" # Append a file.

echo "${multiline_variable}" > "/path/to/outfile.txt"
```


## Loops
Loops are very common, so let's start with some very basic ones.

### Simple Iterator
Just execute a task a fixed number of times.

```bash
for (( i = 1 ; i <= 10 ; i++ )) ; do
    echo -n "${i} "  # 1 2 3 4 5 6 7 8 9 10
done
```

### Lists
Iterate over each item in a list ([source][11]).

```bash
declare -a StringArray=( "Linux Mint" "Fedora" "Red Hat Linux" "Ubuntu" "Debian" )

for val in ${StringArray[@]}; do
   echo $val
done
```

### `while` Loop and Subshells

The `while` loops are a little bit tricky since they usually execute in a
subshell, which means you cannot change variables outside the loop ([source][6])
unless you write it like in the second method here.

```bash
multiline="line 1
line 2"
total_lines=0

echo "${multiline}" | while read line; do
    total_lines=$((total_lines+1))
done
echo "Total: ${total_lines}"  # Total: 0

while read line; do
    total_lines=$((total_lines+1))
done <<< "${multiline}"
echo "Total: ${total_lines}"  # Total: 2
```


## Maths
There is very seldom I write any complex arithmetic in Bash, usually I only do
counters like this.

```bash
i=0
i=$((i+1))
```

For anything more advanced you are recommended to call upon external programs
that handle it much better ([source][7], [source][8]).

Other than that we can check to see if what we have is an integer:

```bash
if ! [[ "${VAR}" =~ ^-?[0-9]+$ ]]; then
    echo "The value '${VAR}' is not recognized as an integer"
fi
```

or perhaps do some basic comparisons ([source][9], [source][10]). Both of
these expressions are the same, but the first one is POSIX compatible.

```bash
if [ "${VAR}" -gt "0" -a "${VAR}" -ne "5" -a "${VAR}" -lt "10" ]; then
    echo "The variable VAR is larger than 0, smaller than 10 but not 5."
fi

if (( ${VAR} > 0 )) && (( ${VAR} != 5 )) && (( ${VAR} < 10 )); then
    echo "The variable VAR is larger than 0, smaller than 10 but not 5."
fi
```


## Files
When checking for files there actually exist quite a lot of options you can
specify beyond the common "file exists" check ([source][12]), but this will
probably be enough in most cases.

```bash
if [ -f "${VAR}" ]; then
    echo "The file '${VAR}' exists."
fi
```

### Files in Directory
One thing I have done is trying to iterate over all `*.conf` files in a
directory. This can be done by either of these two loops, where the `while`
one is a better alternative as it handles filenames with spaces in them
([source][4], [source][5]) and is iterated in alphabetical order.

```bash
for file in /directory/path/*.conf; do
    echo "${file}"
done

while IFS= read -r -d $'\0' file; do
    echo "${file}"
done < <(find /directory/path/ -maxdepth 1 -name "*.conf" -type f -print0 | sort -z)
```

> NOTE: The `while` loop does not use a subshell here so all variables in
> the current scope are mutable ([see previous section](#while-loop-and-subshells)).

### Listening to `inotify` Events
Instead of polling a folder to check for changes, it is possible to utilize the
Linux kernel's inotify feature to react to events as they happen. In the
following example we will wait for a "MOVE_TO" operation to happen in the
Downloads folder, but we are not interested in "*.tmp" files. This could be
useful for waiting for a browser to completing its download (to a temporary
file) and then acting upon the file when it is moved/renamed to its final
name after completion.

> This requires the [`inotify-tools`][13] package to be installed.

```bash
while read -r directory action file; do
    echo "Event in ${directory}: ${action} ${file}"
    # Event in /home/username/Downloads: MOVED_TO file.txt
done < <(inotifywait -m -e moved_to --excludei '^.*?\.tmp$' "~/Downloads")
```






[1]: https://wiki.bash-hackers.org/syntax/pe
[2]: https://www.cyberciti.biz/faq/unix-linux-bash-script-check-if-variable-is-empty/
[3]: https://stackoverflow.com/a/16753536
[4]: https://stackoverflow.com/a/54563899
[5]: https://www.cyberciti.biz/faq/bash-loop-over-file/
[6]: https://stackoverflow.com/a/16854326
[7]: https://www.shell-tips.com/bash/math-arithmetic-calculation/#gsc.tab=0
[8]: https://unix.stackexchange.com/a/40787
[9]: https://wiki.bash-hackers.org/syntax/arith_expr
[10]: https://www.golinuxcloud.com/bash-compare-numbers/
[11]: https://linuxhint.com/bash_loop_list_strings/
[12]: https://tldp.org/LDP/abs/html/fto.html
[13]: https://linux.die.net/man/1/inotifywait
[14]: https://www.gnu.org/software/bash/manual/html_node/The-Set-Builtin.html
[15]: https://linuxize.com/post/how-to-compare-strings-in-bash/
[16]: https://medium.com/mkdir-awesome/case-transformation-in-bash-and-posix-with-examples-acdc1e0d0bc4
