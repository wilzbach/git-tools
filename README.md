Collection of useful scripts for interacting with GitHub
========================================================

- `pr_send` - open a PR from your current branch
  (it will open a temporary editor instance for editing the PR description)
- `pr_push` - push to any PR
- `merge_stable` - merge `upstream/stable` into `upstream/master` and submit a PR
- rebase onto stable
- `pr_split` (alpha) - will automatically split a PR into multiple PRs and submit these (one PR for each file)

How to use
----------

Clone this repository:

```sh
git clone https://github.com/wilzbach/git-tools
```

And then put it in your path:

```sh
export PATH="$(pwd)/git-tools:$PATH"
```

Required dependencies
---------------------

- `bash`
- `curl`
- [`jq`](https://stedolan.github.io/jq/)

Optional dependencies
---------------------

- [`hub`](https://github.com/github/hub) - allows to directly open the PR
- `upstream` remote endpoint

Most of these scripts need an `upstream` endpoint to work correctly.
Set it e.g. by:

```
git remote add upstream git@github.com:dlang/phobos.git
```

Clone any PR
------------

### Checkout a single PR

Set the `pr` alias once for your global `.gitconfig`:

```shell
git config --global alias.pr '!f() { git fetch -fu ${2:-$(git remote |grep ^upstream || echo origin)} refs/pull/$1/head:pr/$1 && git checkout pr/$1; }; f'
```

Now fetch and checkout a single PR:

```shell
git pr 123
```

### Fetch all PRs

Alternatively you can also add an additional `fetch = +refs/pull/*/head:refs/remotes/origin/pr/*`
line in your `.git/config` file:

```shell
[remote "upstream"]
	url = git@github.com:dlang/phobos.git
	fetch = +refs/heads/*:refs/remotes/origin/*
	fetch = +refs/pull/*/head:refs/remotes/origin/pr/*
```

Now fetch `upstream` as usual:

```shell
git fetch upstream
```

And switch to any PR:

```
git checkout pr/123
```

`pr_send` - open a PR from your current branch
----------------------------------------------

```bash
git checkout -b my-fancy-branch
# do some work
pr_send
```

- it will open a temporary editor instance for editing the PR description
- opens a new PR against `upstream/master` or `origin/master`

`pr_push` - push to any PR
--------------------------

```bash
git pr 123
# do some work
pr_push
```

__Warning__: This tool with force-push to the PR. Be careful. You have been warned.

Merge any PR onto stable
------------------------

You can combine `pr_push` to rebase any PR to `upstream/stable`:

```bash
git fetch upstream
git pr 6060
git rebase --onto upstream/stable upstream/master
pr_push
```

`merge_stable` - merge `upstream/stable` into `upstream/master`
---------------------------------------------------------------

- execute within the current git repo or pass the git repository as first argument

```bash
merge_stable
```

`pr_split`: Submitting step-by-step
-----------------------------------

The first command will show you all the files that are considered as changes:

```sh
pr_split
```

Without any argument it will _simulate_ the split and don't do any action:

```
Adding: public/content/de/welcome/links-documentation.md
Adding: public/content/de/welcome/run-d-program-locally.md
Simulation was run. Now use -f/--force to apply.
```

Accordingly to submit these files to Github, use `-f` or `--force`:

```sh
pr_split -f
```

By default the tool will only process one PR per run, however once you are familiar
with the tool you can enable the batch mode with `-a` or `--all`.

```sh
pr_split -f --all
```

_Warning_:  If you run this script for the first time, make a copy of your work beforehand.

### Usage with an existing commit or branch

#### A) With a custom branch

Change to your master branch and checkout all changes from your branch:

```
git checkout master
git checkout my-fancy-branch -- .
```

#### B) With an commit on `master`

Simply reset your changes to the ones of the `upstream` branch.
Warning: this will remove your new commits, but not the changes.

```
git reset --soft upstream/master
```

If you haven't an `upstream` remote, you can also try to remove the commits directly.
For example to remove one commit, but keep the changes run:

```
git reset --soft HEAD~1
```

### Options

#### `-p`, `--prefix`

To categorize the PRs the `-p` or `--prefix` flag can be given. It will
be prefixed before every PR and commit message. Example:

```sh
pr_split -p "[german]"
```

#### `-m`, `--message`

Similar to `-p`, but applied only for the git commit message.
It comes after the prefix, but before the file name.

```sh
pr_split -m "My fancy git commit message"
```

#### `-b`, `--branchprefix`

Allows to use a prefix before the generated branches, e.g.

```sh
pr_split -b "run_spec_"
```

File paths will use underscores instead of slashes.

#### `--basename`

For all added files, only use their `basename` for the branch name and
git commit message.
By default, the path within the git directory is used, but with slashes
translated to underscores. Example

- file: `foo/bar/duck.md`
- default branch name: `foo_bar_duck`
- `--basename` branch name: `duck`

Be careful with this option.
Multiple files with the same filename will lead to errors.

#### `-f`, `--force`

Switch from the simulation mode and do the actions in the real world.

#### `-a`, `--all`

Don't stop after sending one PR, send all in one run.

### Full example

#### 1) Simulation

```sh
pr_split -m 'Allow examples of the D spec to be runnable:' -b "run_spec_" -p "[run-spec]"
```

#### 2) Step by step

```sh
pr_split -m 'Allow examples of the D spec to be runnable:' -b "run_spec_" -p "[run-spec]" -f
```

After one or two PRs, you can use `--all` to submit all remaining files.

[See the result](https://github.com/dlang/dlang.org/pulls?utf8=%E2%9C%93&q=%5Brun-spec%5D).

Help
----

Let me know if you run into [issues](https://github.com/wilzbach/splitup-prs/issues).

License
-------

Boost Software License - Version 1.0 - August 17th, 2003

Permission is hereby granted, free of charge, to any person or organization
obtaining a copy of the software and accompanying documentation covered by
this license (the "Software") to use, reproduce, display, distribute,
execute, and transmit the Software, and to prepare derivative works of the
Software, and to permit third-parties to whom the Software is furnished to
do so, all subject to the following:

The copyright notices in the Software and this entire statement, including
the above license grant, this restriction and the following disclaimer,
must be included in all copies of the Software, in whole or in part, and
all derivative works of the Software, unless such copies or derivative
works are solely in the form of machine-executable object code generated by
a source language processor.

THE SOFTWARE IS PROVIDED "AS IS", WITHOUT WARRANTY OF ANY KIND, EXPRESS OR
IMPLIED, INCLUDING BUT NOT LIMITED TO THE WARRANTIES OF MERCHANTABILITY,
FITNESS FOR A PARTICULAR PURPOSE, TITLE AND NON-INFRINGEMENT. IN NO EVENT
SHALL THE COPYRIGHT HOLDERS OR ANYONE DISTRIBUTING THE SOFTWARE BE LIABLE
FOR ANY DAMAGES OR OTHER LIABILITY, WHETHER IN CONTRACT, TORT OR OTHERWISE,
ARISING FROM, OUT OF OR IN CONNECTION WITH THE SOFTWARE OR THE USE OR OTHER
DEALINGS IN THE SOFTWARE.
