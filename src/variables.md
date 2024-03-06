# Variables

Make supports variable assignment. All variables in make are strings:
```makefile
my_variable = this is the content
```
You can then reference the text in the variable by using `$(VAR)`:
```makefile
my_variable = this is the content

default:
	echo "$(my_variable)"
```
To use the variable, make will **literally** remove `$(VAR)` and put the string
inside the variable in its place.

> This is important to remember.
> Make **literally** substitutes the string in place of the variable, without
> any magic.

The above file is *exactly the same* as this one:
```makefile
default:
	echo: "this is the content"
```

Make first reads the *whole* makefile, then substitutes the variables, and then
executes the rules.

> You will have noticed that the `$(VAR)` syntax is also used for shell variable
> substitutions.
> If you want to use *shell* variables in your rules, you will need to escape the
> `$` used by make; e.g. use `$$(SHELL_VAR)` instead of `$(VAR)`.
> While expanding variables, make will convert `$$` to `$`, and pass it to the
> shell, therefore making it valid.

Make copies **all** environment variables upon starting as if they where
written in the makefile.
For example, this works:
```makefile
path_content.txt:
	# Notice that the variable here is a make variable, since it has just
	# one \$ not two.
	echo $(PATH) > path_content.txt
```

You can conditionally override environment variables with the `?=` assignment.
This assigns the value of the variable *only* if it's not already assigned:
```makefile
some_var ?= my_text

default:
	echo $(some_var)
```
If you run `make`, the output will be `"my text"`.
If you run `some_var="alternative text"`, the output will be `"alternative text"`,
since the assignment will not be made.

> Try to keep your variables [`snake_case`](https://en.wikipedia.org/wiki/Snake_case).

## What to do with variables
You can do a lot with variables.
They are most commonly used to write the requirements for a rule:
```makefile
files = one.txt two.txt three.txt

output: $(files)
    ...
```
You can also use them to shorten long calls:
```makefile
e_dir = long/path/to/executable/directory
flags = --some --default --flags --that-are --always-used

output:
	$(e_dir)/create_file $(flags) > output
```

> This is very useful to work with stuff like `Rscript`:
> ```makefile
> $(r) = Rscript --vanilla
> output:
> 	$(r) my_r_script.R > output
> ```

You can use them for variables that the user can override:
```makefile
option ?= default_value

default:
	execute --var $(option)
```
The user can use the default value or alternatively export a `option` variable
and override it.

## Automatic variables
When a recipe is run, make sets some automatic variables, so that you can
write your recipes in a less verbose way.

You can read the [list of all automatic variables](https://www.gnu.org/software/make/manual/html_node/Automatic-Variables.html)
in the make manual.

> [!NOTE]
> Variables just one character long - like most automatic variables - may omit
> the `()` wrapping the variable name. E.g. `$(@)` is the same as `$@`. 

Here are some of the most commonly used ones:

`$@` is the target of the rule:
```makefile
path/to/target.txt: requirement.txt
	cat requirement.txt > $@
	# cat requirement.txt > path/to/target.txt
```

`$<` is the *first* requirement. Careful to use this when you have more than
one requirement:
```makefile
path/to/target.txt: requirement.txt
	cat $< > $@
	# cat requirement.txt > path/to/target.txt
```

`$(@D)` and `$(@F)` are the directory of the target file and the name of the
target file, respectively.
This is very useful to create containing folders for output files:
```makefile
path/to/target.txt: requirement.txt
	mkdir -p $(@D)
	# mkdir -p path/to/
	cat $< > $@
	# cat requirement.txt > path/to/target.txt
```
You can do the same with `$(<D)` and `$(<F)`.

`$^` is the *full* list of requirements, separated by spaces:
```makefile
target.txt: one.txt two.txt three.txt
	concat_files $^
	# concat_files one.txt two.txt three.txt
```

> You may wonder how to select the N-th requirement.
> There is no automatic variable for each requirement, but you can use `$^` to
> get it with the `word` function: `$(word n, $^)` where `n` is the 1-indexed
> position of the requirement you want.
> For example, if `one two three` are the requirements, `$(word 2, $^)` will
> result in the string `two`.
>
> Read more about functions [here](functions.md) or
> [in the manual](https://www.gnu.org/software/make/manual/html_node/Functions.html).

