# DVC Files and Directories

Once initialized in a <abbr>project</abbr>, DVC populates its installation
directory (`.dvc/`) with the internal files and directories needed for DVC
operation:

- `.dvc/config`: This is a configuration file. The config file can be edited by
  hand or with the `dvc config` command.

- `.dvc/config.local`: This is a local configuration file, that will overwrite
  options in `.dvc/config`. This is useful when you need to specify private
  options in your config that you don't want to track and share through Git
  (credentials, private locations, etc). The local config file can be edited by
  hand or with the command `dvc config --local`.

- `.dvc/cache`: The [cache directory](#structure-of-cache-directory) will store
  your data. The data files and directories in DVC repositories will only
  contain links to the data files in the cache. (Refer to
  [Large Dataset Optimization](/docs/user-guide/large-dataset-optimization). See
  `dvc config cache` for related configuration options.

  > Note that DVC includes the cache directory in `.gitignore` during
  > initialization. No data tracked by DVC will ever be pushed to the Git
  > repository, only [DVC-files](/doc/user-guide/dvc-file-format) that are
  > needed to download or reproduce them.

- `.dvc/state`: This file is used for optimization. It is a SQLite database,
  that contains checksums for files tracked in a DVC project, with respective
  timestamps and inodes to avoid unnecessary checksum computations. It also
  contains a list of links (from cache to <abbr>workspace</abbr>) created by DVC
  and is used to cleanup your workspace when calling `dvc checkout`.

- `.dvc/state-journal`: Temporary file for SQLite operations

- `.dvc/state-wal`: Another SQLite temporary file

- `.dvc/updater`: This file is used store the latest available version of DVC.
  It's used to remind the user to upgrade when the installed version is behind.

- `.dvc/updater.lock`: Lock file for `.dvc/updater`

- `.dvc/lock`: Lock file for the entire DVC project

## Structure of cache directory

There are two ways in which the data is stored in <abbr>cache</abbr>. It depends
on whether the data in question is a single file (eg. `data.csv`) or a directory
of files.

For the first case, we calculate the file's checksum, a 32 characters long
string (usually MD5). The first two characters are used to name the directory
inside `.dvc/cache` and the rest become the file name of the cached file. For
example, if a data file `Posts.xml.zip` has checksum
`ec1d2935f811b77cc49b031b999cbf17`, its cache entry will be
`.dvc/cache/ec/1d2935f811b77cc49b031b999cbf17` locally. If pushed to
[remote storage](/doc/command-reference/remote), its location will be
`<prefix>/ec/1d2935f811b77cc49b031b999cbf17`, where prefix is the name of the
DVC remote.

For the second case, let us consider a directory with 2 images.

```dvc
$ tree data/images/
data/images/
├── cat.jpeg
└── index.jpeg

$ dvc add data/images
...
```

When running `dvc add` on this directory of images, a `data/images.dvc`
[DVC-file](/doc/user-guide/dvc-file-format) is created, containing the checksum
of the directory:

```yaml
md5: 77e511dafe2178d936e54331d5d6288f
outs:
  - md5: 196a322c107c2572335158503c64bfba.dir
    path: data/images
    # ...
```

The directory in cache is stored as a JSON metafile describing it's contents,
along with the files it contains in cache, like this:

```dvc
$ tree .dvc/cache
.dvc/cache/
├── 19
│   └── 6a322c107c2572335158503c64bfba.dir
├── d4
│   └── 1d8cd98f00b204e9800998ecf8427e
└── 20
    └── 0b40427ee0998e9802335d98f08cd98f
```

The cache file with `.dir` extension is a special text file that stores the
mapping of files in the `data/` directory (as a JSON array), along with their
checksums. The other two cache files are the files inside `data/`. A typical
`.dir` cache file looks like this:

```dvc
$ cat .dvc/cache/19/6a322c107c2572335158503c64bfba.dir
[{"md5": "dff70c0392d7d386c39a23c64fcc0376", "relpath": "cat.jpeg"},
{"md5": "29a6c8271c0c8fbf75d3b97aecee589f", "relpath": "index.jpeg"}]
```

See also `dvc cache dir` to set the location of the cache directory.
