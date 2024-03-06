# Pattern matching

To convert a `.tsv` to a `.csv` file, you can use the [`xsv`](https://github.com/BurntSushi/xsv)
tool, with the command:

```bash
xsv fmt -d '\t' file.tsv > file.csv
```

We can write the same in our makefile:
```makefile
file.csv: file.tsv
	xsv fmt -d '\t' file.tsv > file.csv
```

However, since we can do this for *any* `.tsv` file, we can write a *generic rule*:
```makefile
%.csv: %.tsv
	xsv fmt -d '\t' $< > $@
```
The `%` is a wildcard, working similarly to a shell `*`.
In this case, any file ending in `.csv` can be created from a file (in the same
folder) ending in `.tsv`:
- `path/to/file.csv` from `path/to/file.tsv` -> OK!;
- `file.csv` from `path/to/file.csv` -> NO! The files do not share the same stem;
- `file.csv` from `file.tsv.gz` -> NO! The files do not share the same suffix;

> [!NOTE]
> Here, using automatic variables is not optional.
> Since we do not know what '%' will be replaced with at runtime.

Make will use the generic rule whenever it *has* to, but not more than that.
This means that you cannot have make create *all possible* `.csv` files from
all `.tsv` files.
You need to specifically ask for `.csv` files as requirements to have make
create them.
Assume that you have the `one.tsv` and `two.tsv` files and you want to convert
them in `.csv`.
You could write this makefile:

```makefile
default: one.csv two.csv
# The above rule has no body, but it's ok! Make will just do nothing when
# the rule is executed.

%.csv: %.tsv
	xsv fmt -d '\t' $< > $@
```

When you call `make`, it will use the generic rule twice to create the
prerequisites of `default`, as if you had written:
```makefile
default: one.csv two.csv

one.csv: one.tsv
	xsv fmt -d '\t' $< > $@

two.csv: two.tsv
	xsv fmt -d '\t' $< > $@
```
