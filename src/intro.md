# Introduction

Make creates files based on recipes, know in Make jargon as ***rules***.
By default, `make` looks for and reads a file in the current working
directory named `makefile`.

> You can tell `make` to read another file with `make -f path/to/makefile`.
> Note that all paths in a makefile are relative to the working directory
> in which `make` was called, not where the `makefile` is.

A *make* rule looks like this:

```makefile
file/to/create:
	echo "Commands to create the file" > file/to/create
```

Let's break it down.
The first line is the rule's ***signature***.
`file/to/create:` tells make that this rule creates a file named  `create` in
the folder `file/to/`.
This file is the rule's ***target***.
It's up to you to *actually* write commands that create this file:
make does not check that the file is really created after a rule is run.
This *can* be useful in some cases, as we will see later.

Next, we have the ***body*** of the rule (e.g. the `echo ...` line).
It MUST be indented with a \<Tab\> (`\t`) directly underneath the rule's signature.
Each line that starts with a \<Tab\> thereafter is part of the same rule:

```makefile
target:
	command # Body starts here
	        # |
    command # |
	        # Last line of the body
# Here the rule has ended - there is no Tab
```
Each line in the body of a rule is a shell command that will be run when
the rule is executed.

You can specify ***requirement*** files in a rule's signature after the `:`:

```makefile
file/to/create: first_requirement second_requirement
	wc -l first_requirement >> file/to/create
	wc -l second_requirement >> file/to/create
```

This lets Make know that to create `file/to/create` you first need the
`first_requirement` and `second_requirement` files.
It's here that makes becomes useful: it can "string together" different rules
to create the files that we want:

```makefile
output_file: intermediate_file
	wc intermediate_file > output_file

intermediate_file:
	echo "This is some words in the file" > intermediate_file
```

> You need not write rules for all requirement files.
> If make finds a requirement with no rule, and the file is not already
> present in the filesystem, it simply fails with an error.

Make sees the `output_file` as the first rule. It implicitely sets it to be the
***default target*** and tries to create it.
It sees that it first needs to create the `intermediate_file`.
It has a rule for it, so it executes that rule first, followed by the rule
for the `output_file`. Done!

Internally, make creates a dependency graph of the rules, starting from the
default target(s). For example, consider this `makefile` (rule bodies are omitted
for clarity):

```makefile
Target: req1 req2
	# ...

req2: sub_req_1 sub_req_2
	# ...

sub_req_1:
	# ...
```

It will be parsed to a tree structure like this:

```
           ┌─────┐               
         ┌►│req 1│               
┌──────┐ │ └─────┘    ┌─────────┐
│Target├─┤         ┌─►│sub req 1│
└──────┘ │ ┌─────┐ │  └─────────┘
         └►│req 2├─┤             
           └─────┘ │  ┌─────────┐
                   └─►│sub req 2│
                      └─────────┘
```

From this, `make` can then run the rules in the correct order: from the "leaves"
of the tree up to the "root".
You can also see that there is no relationship between the `req 2` and `req 1`
branches.
If you tell `make` to run in parallel (with the `-j` or `--jobs` flag), `make`
will run these independent branches in parallel for you, speeding up execution
by a lot.
See the [parallel execution section of the manual](https://www.gnu.org/software/make/manual/html_node/Parallel.html)
to learn more.

Another useful feature of make is that it *skips creating files that are already there*.
In the example above, say that `sub req 2` changed, but everything else did not.
Make is smart enough to only run the rule for `sub req 2` (if any), then just
`req 2` and then `Target`, since that branch is out of date:

```
           ┌─────┐               
         ┌►│req 1│               
┌ ─ ─ ─┐ │ └─────┘    ┌ ─ ─ ─ ─ ┐
 Target ─┤         ┌─► sub req 1 
└─ ─ ─ ┘ │ ┌ ─ ─ ┐ │  └ ─ ─ ─ ─ ┘
         └► req 2 ─┤             
           └ ─ ─ ┘ │  ┌─────────┐
                   └─►│sub req 2│
                      └─────────┘
```

You can now hopefully see why writing makefiles instead of shell scripts for
complex file transformations is useful.
