# Weirdness of Makefiles

This section covers a bunch of edge cases that you will encounter as you write
more complex makefiles.

## Unknown prerequisites
Assume you have a `create_random_files` function that does just that: it makes
some `N` number of files, with unknown file names, in some output directory.

You want to use all of these files as input to create a `all.txt` file, just like
the above example.
You might write this makefile (that will **not** work):

```makefile
files = $(wildcard input/*.txt)

all.txt: $(files)
	merge_files $^

create_files:
	create_random_files input/

.DEFAULT_TARGET: all.txt create_files
```
We cannot control in *what order* make will run our rules.
Assuming `input/` is originally empty, the `all.txt` requirements might be
evaluated to be... nothing, since `create_files` has not been run yet.

Even if the above issue would not occur, we would have other problems.
In this makefile, the `create_files` rule will be always run, since it's
a ([phony](special_variables.md)) default target.
This causes `all.txt` to also be always re-run, since its requirements are always
newer than the target.
If `create_files` is near the start of our make graph, it triggers the remake
of many (if not all) rules, which defeats the purpose of using make.

There is no elegant way to fix this issue.
However, we *can* fix it by using a **flag file**.
These files are not used in the make process, but are just there to serve as
timestamps of creation of other - unknown - files:

```makefile
files = $(wildcard input/*.txt)

all.txt: flags/create_files.flag
	merge_files $(files)

flags/create_files.flag:
	create_random_files input/
    touch $@
```

The `flags/create_files.flag` file is empty, but has a timestamp record of
when its rule is run (by the `touch` command).
We can then use it in place of the `$(files)` variable when we define the
requirements for a rule. 

In this way, the rules will be executed in the correct order, so `$(files)` will
always be correctly populated, plus we do not lose the ability of make to only
recreate out-of-date files.

It's always a good idea to keep flag files to a minimum.
**Rules that create or act upon an unknow number of files should be rare**.
Using flag files can quickly become a crutch that allows your scripts to take
a *folder* as input instead of a list of files to be processed, but this will
bite you in the ass in the long-run, or when you need to process one specific
file instead of a bunch of them.
