# get

Download or copy file or directory from any <abbr>DVC project</abbr> in a Git
repository (e.g. hosted on GitHub) into the current working directory.

> Unlike `dvc import`, this command does not track the downloaded data files
> (does not create a DVC-file).

## Synopsis

```usage
usage: dvc get [-h] [-q | -v] [-o [OUT]] [--rev [REV]] url path

positional arguments:
  url         URL of Git repository with DVC project to download from.
  path        Path to data within DVC repository.
```

## Description

Provides an easy way to download datasets, intermediate results, ML models, or
other files and directories (any <abbr>data artifact</abbr>) tracked in another
DVC repository, by downloading them into the current working directory.

Note that this command doesn't require an existing DVC project to run in. It's a
single-purpose command that can be used out of the box after installing DVC.

The `url` argument specifies the address of the Git repository containing the
external <abbr>project</abbr>. Both HTTP and SSH protocols are supported for
online repositories (e.g. `[user@]server:project.git`). `url` can also be a
local file system path to an "offline" repository.

The `path` argument of this command is used to specify the location of the data
to be downloaded within the source project. It should point to a data file or
directory tracked by that project – specified in one of the
[DVC-files](/doc/user-guide/dvc-file-format) of the repository at `url`. (You
will not find these files directly in the source Git repository.) The source
project should have a default [DVC remote](/doc/command-reference/remote)
configured, containing them.)

> See `dvc get-url` to download data from other supported URLs.

After running this command successfully, the data found in the `url` `path` is
created in the current working directory, with its original file name.

## Options

- `-o`, `--out` - specify a path (directory and/or file name) to the desired
  location to place the imported data in. The default value (when this option
  isn't used) is the current working directory (`.`) and original file name. If
  an existing directory is specified, then the output will be placed inside of
  it.

- `--rev` - specific Git revision of the DVC repository to import the data from.
  The tip of the default branch is used by default when this option is not
  specified.

- `-h`, `--help` - prints the usage/help message, and exit.

- `-q`, `--quiet` - do not write anything to standard output. Exit with 0 if no
  problems arise, otherwise 1.

- `-v`, `--verbose` - displays detailed tracing information.

## Examples

> Note that `dvc get` can be used from anywhere in the file system, as long as
> DVC is [installed](/doc/get-started/install).

We can use `dvc get` to download the resulting model file from our
[get started example repo](https://github.com/iterative/example-get-started), a
<abbr>DVC project</abbr> external to the current working directory. The desired
<abbr>output</abbr> file would be located in the root of the external project
(if the
[`train.dvc` stage](https://github.com/iterative/example-get-started/blob/master/train.dvc)
was reproduced) and named `model.pkl`.

```dvc
$ dvc get https://github.com/iterative/example-get-started model.pkl
$ ls
model.pkl
```

Note that the `model.pkl` file doesn't actually exist in the
[root directory](https://github.com/iterative/example-get-started/tree/master/)
of the external Git repository. Instead, the corresponding DVC-file
[train.dvc](https://github.com/iterative/example-get-started/blob/master/train.dvc)
is found, that specifies `model.pkl` in its outputs (`outs`). DVC then
[pulls](/doc/command-reference/pull) the file from the default
[remote](/doc/command-reference/remote) of the external DVC project (found in
its
[config file](https://github.com/iterative/example-get-started/blob/master/.dvc/config)).

> A recommended use for downloading binary files from DVC repositories, as done
> in this example, is to place a ML model inside a wrapper application that
> serves as an [ETL](https://en.wikipedia.org/wiki/Extract,_transform,_load)
> pipeline or as an HTTP/RESTful API (web service) that provides predictions
> upon request. This can be automated leveraging DVC with
> [CI/CD](https://en.wikipedia.org/wiki/CI/CD) tools.

The same example applies to raw or intermediate <abbr>data artifacts</abbr> as
well, of course, for cases where we want to download those files or directories
and perform some analysis on them.

## Example: Compare different versions of data or model

`dvc get` has the `--rev` option, to specify which version of the repository to
download a <abbr>data artifact</abbr> from. It also has the `--out` option to
specify the file or directory path and file name for the download. Combining
these two options allows us to do something we can't achieve with the regular
`git checkout` + `dvc checkout` process – see for example the
[Get Older Data Version](/doc/get-started/older-versions) chapter of our _Get
Started_ section.

Let's use the
[get started example repo](https://github.com/iterative/example-get-started)
again, like in the previous example. But this time, clone it first to see
`dvc get` in action inside a <abbr>DVC project</abbr>.

```dvc
$ git clone git@github.com:iterative/example-get-started.git
$ cd example-get-started
```

If you are familiar with our [Get Started](/doc/get-started) example, you may
know that each chapter has a corresponding
[tag](https://github.com/iterative/example-get-started/tags). Tag `7-train` is
where we train a first version of the example model, and tag `9-bigrams-model`
has an improved model (trained using bigrams). What if we wanted to have both
versions of the model "checked out" at the same time? `dvc get` provides an easy
way to do this:

```dvc
$ dvc get . model.pkl --rev 7-train --out model.monograms.pkl
```

> Notice that the `url` provided to `dvc get` above is `.`. `dvc get` accepts
> file system paths as a "URL" to the repository to get the data from for edge
> cases.

The `model.monograms.pkl` file now contains the older version of the model. To
get the most recent one, we use a similar command, but with

`-o model.bigrams.pkl` and `--rev 9-bigrams-model` or even without `--rev`
(since it's the latest version anyway). In fact, in this case using `dvc pull`
with the corresponding [DVC-files](/doc/user-guide/dvc-file-format) should
suffice, downloading the file as just `model.pkl`. We can then rename it to make
its version explicit:

```dvc
$ dvc pull train.dvc
$ mv model.pkl model.bigrams.pkl
```

And that's it! Now we have both model files in the <abbr>workspace</abbr>, with
different names, and not currently tracked by Git:

```dvc
$ git status
...
Untracked files:
  (use "git add <file>..." to include in what will be committed)

	model.bigrams.pkl
	model.monograms.pkl
```
