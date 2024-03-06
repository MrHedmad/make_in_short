# Introduction

Make creates files based on recepies, know in make jargon as "rules".
You write rules that make has to follow in the `makefile` file.
A rule looks like this:

```makefile
file/to/create:
	echo "Commands to create the file" > file/to/create
```

Let's break it down:

```makefile
file/to/create: # This line tells make that this rule creates 'file/to/create'
	# This is the body of the rule. It MUST be indented with a <Tab> (\t),
	# or make won't know this is the body of the rule.
	echo "Commands to create the file" > file/to/create
```

It's up to you to actually create the file that you specify will be created.
Make does not check that the file is *actually* created.

You can specify **requirement** files in a rule after the `:`:

```makefile
file/to/create: first_requirement second_requirement
	wc -l first_requirement >> file/to/create
	wc -l second_requirement >> file/to/create
```

This lets Make know that to create the `file/to/create` you first need the
`first_requirement` and `second_requirement` files.
It's here that makes becomes useful: it can "string together" different rules
to create the files that we want:

```makefile
output_file: intermediate_file
	wc intermediate_file > output_file

intermediate_file:
	echo "This is some words in the file" > intermediate_file
```

Make sees the `output_file` as the first rule. It implicitely sets it to be the
**default target** and tries to create it.
It sees that it first needs to create the `intermediate_file`.
It has a rule for it, so it executes that rule first, followed by the rule
for the `output_file`. Done!

The useful feature of make is that it *skips creating files that are already there*.
If the `intermediate_file` already exists, make will not recreate it.
It also looks at file modification timestamps: if `output_file` is older than
`intermediate_file`, it probably needs to be recreated, so make executes the
rule for `output_file` again.

