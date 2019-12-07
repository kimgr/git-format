# git-format

[`git-clang-format`][1] is wonderfully feature complete, but I've found it
clunky to use. Rather than try and learn it properly or bend it to my will, I've
just built a simple replacement focused on my workflows: `git-format`.

It has two major use cases --

* Reformat an arbitrary git diff
* Reformat working tree, index or HEAD in place

With these two atoms, I've found that I can easily automate format checking and
reformatting.

[1]: https://github.com/llvm-mirror/clang/blob/master/tools/clang-format/git-clang-format


## Recipes ##

### Check formatting of a patch set ###

Say you have a branch `topic`, and you want to see if there are any formatting
problems in the commits between it and `master`. This boils down to:

    $ git diff -U0 master..topic | git-format
    $ echo $?
    0

Should there be any reformatting required, `git-format` will print the formatted
diff and return a non-zero exit code.

### Reformat working tree ###

You've made code changes using an editor that does not automatically format on
save. Use:

    $ git-format

or

    $ git-format --unstaged

to reformat all modified files in place.

### Reformat index ###

You've staged your changes and you're ready to commit. But oh!, you forgot to
format! Use:

    $ git-format --staged

or

    $ git-format --cached

to reformat the index and modify all affected files in your working tree. If
you're happy with the changes, say `git add -u` to squash the changes into the
index.

### Reformat HEAD ###

Oops! You forgot to format before committing. Use:

    $ git-format --amend

to reformat the latest commit and amend it in place. This rewrites history, so
be careful with shared branches.

### Fixup HEAD ###

Oops! You forgot to format before committing, AGAIN. Use:

    $ git-format --fixup

to reformat the latest commit and create a fixup commit with the formatting
changes.

### Reformat every commit on a branch ###

You've gone wild and committed a series of badly formatted changes on a topic
branch. Now you want these changes reviewed, and need to clean them up
fast. Use:

    $ git rebase -i master..topic --exec "git-format --amend"

to reformat every commit in place.

This is a little risky -- if the formatter makes a destructive mistake, your
changes are gone from the branch (but there's always `git reflog`).

### Fixup every commit on a branch ###

Same scenario, but the safer option. Use:

    $ git rebase -i master..topic --exec "git-format --fixup"

to create a reformatted fixup commit for every commit on the branch (only
reformatted commits get fixups).

This lets you review the formatting before you commit to it. Once you're happy:

    $ git rebase -i --autosquash master..topic

to squash the fixup commits into the original commits.


## Configuration ##

If different repos or directories in your repo use different `clang-format`
versions, for whatever reason, you can configure the path of the `clang-format`
binary in a simple `ini`-style config file called `.git-format`:

    [format]
    binary=/usr/lib/llvm-4.0/bin/clang-format

`git-format` uses a strategy similar to `clang-format` when searching for
configuration:

* For every file being formatted, search the directory tree upwards all the way
  to the Git topdir looking for `.git-format`
* If not found, try `~/.git-format`

The default, if no config is found, is to use `clang-format` on the system
`PATH`.


## Caveat ##

Note that most commands are potentially destructive, as they change content in
place.

The exceptions are:

* `git diff -U0 | git-format`, which only prints the diff reformatted
* `git-format --staged`, which applies changes to the working tree, so you can
  review them before squashing them into the index
* `git-format --fixup`, which generates distinct fixup commits, so you can
  review them before squashing them into the actual commits

Be particularly careful when launching `git-format` without arguments, it will
reformat your working directory in place without asking.

All other commands are non-destructive or warn if you have unstaged changes.
