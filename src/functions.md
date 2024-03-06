# Functions

Make has support for a ton of text and path parsing built-in functions.

Confusingly, functions are called just as variables are referenced, with the
`$(fun arg1,arg2,...,argn)` syntax: the name of the function, a space, and a
comma-separated list of arguments.

> You cannot escape characters in functions.
> [Read more](https://www.gnu.org/software/make/manual/html_node/Syntax-of-Functions.html)

Functions can do a *lot* of things. They are where the real power of make is.
The full list of functions can be found in [the manual](https://www.gnu.org/software/make/manual/html_node/Functions.html).

You can do a bunch of things with functions. Here are some examples.

### Finding files

The `wildcard` function searches for files:
```makefile
files = $(wildcard inputs/*.txt)

default:
	echo "Input files are: $(files)"
```

> Please read the [more about variables](variable_weirdness.md) section
> before using the `wildcard` function!

This is very useful for finding requirements.
Say that the requirement for a `all.txt` file are *all* the files in the `input`
folder, but you don't know in advance what the `input` folder will contain.
You can use the `wildcard` function for that:

```makefile
files = $(wildcard input/*)

all.txt: $(files)
	merge_files $^
```
