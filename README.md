# bash_tips_and_tricks

I don't write Bash code that often, so when I need to do it I usually have to
Google the same stuff every time because I cannot remember exactly how it should
be done. So in order to reduce the time wasted on remembering what search term
I used the last time I found the best guide I will just collect them here (with
links back to the source of course).

I think I will keep everything in a single document so it will be easier to
`Ctrl+F` stuff.


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

### Files in Directory
One thing I have done is trying to iterate over all `*.conf` files in a
directory. This can be done by either of these two loops, where the `while`
one is a better alternative as it handles filenames with spaces in them
([source][4], [source][5]).

```bash
for file in /directory/path/*.conf; do
    echo "${file}"
done

while IFS= read -r -d $'\0' file; do
    echo "${file}"
done < <(find /directory/path/ -name "*.conf" -maxdepth 1 -type f -print0)
```

> NOTE: The `while` loop does not use a subshell here so all variables in
> the current scope are mutable ([see next section](#while-loop-and-subshells)).

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
