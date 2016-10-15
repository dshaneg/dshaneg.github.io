---
layout: post
title: git mergetool setup with p4merge et al
---

Here's how to get rolling with a good visual merge tool for git.

If you are using Visual Studio, then you already have a great tool at your disposal--that venerable behemoth pretty much has everything sewn up.
However, if you've ventured outside that comfortable old tool, and you're using a light-weight editor like
[Visual Studio Code](https://code.visualstudio.com), you'll probably find yourself
a little bit frustrated when you get ready to merge some code and git tells you there are conflicts.

Sure, you could just open up those files and grind through the conflict markers [like a boss](https://help.github.com/articles/resolving-a-merge-conflict-from-the-command-line/),
but if you're like me, you'll probably want something a good bit more helpful.

Enter the visual merge tool. There are quite a few out there that have garnered a fair sized fan base.
Here's a short list.

| tool | notes
| --- | --- |
| [KDiff3](https://sourceforge.net/projects/kdiff3/files/) | built in linux for linux; ported to windows as kind of an afterthought.
| [DiffMerge](https://sourcegear.com/diffmerge/downloads.php) | built by the sourcegear folks; I used this one fairly heavily for a while.
| [p4merge](https://www.perforce.com/downloads/helix#product-10) | part of the Perforce distribution, but git doesn't mind if you use tools from another SCM. To get to the bits from that link, scroll down to ***Helix P4V: Visual Client***. When the installer asks which applications you want, turn off everything but p4merge.

Any of these tools will work great for you, but I like p4merge the best nowadays. The UI takes a few minutes to understand, but
this seems to be the most up-to-date tool in the list, and I like the 4-pane view. (It shows you the local file, the file you're merging from,
the common ancestor file, and on the bottom, a pane where you do the merging.

If you go searching, the web will turn up all kinds of hits that will tell you how to run `git config --global blah blah blah` to set up
your external diff and merge tools for git. The command line is great, but in this case, I think it's easier to just open the `.gitconfig` file
and start typing.

## editing .gitconfig for fun and profit

The `.gitconfig` file lives in your home directory (I'm going to assume you're running Windows here). So if you open up
`C:/Users/yourusername`, you'll see a `.gitconfig` file sitting there (I'm also assuming you already have git installed). Open that file with
your favorite text editor. You'll notice that this thing looks like a plain old-fashioned `.ini` file, because it pretty much is just that.

I'm going to describe the sections we need to add in the reverse order of how you most often see them in the file.
Seems dumb, but this order makes more sense to me in understanding how the file works.

To set up your external merge tool, you'll need to add some lines. First, we'll declare the tool with

```ini
[mergetool "p4merge"]
  cmd = C:/Program\\ Files/Perforce/p4merge.exe \"$BASE\" \"$LOCAL\" \"$REMOTE\" \"$MERGED\"
  trustExitCode = false
```

This section defines the tool's name and tells git how to call it. You'll also notice that we can't trust it to give us a valid exit code.
That's OK. It doesn't mean the tool sucks; it just means somebody got lazy. (DiffMerge does not shirk this responsibility BTW)

The `.gitconfig` file allows you to declare as many mergetools as you want. All you need to do is give each one a different name in the
section header. *More on this later...*

Now that we've defined the tool, let's tell git that we want to use it. Let's add another section.

```ini
[merge]
  tool = p4merge
```

Notice that in the merge section, we just need to list the name we gave our merge tool as *the* tool, and we're off to the races.
Now when you get a merge conflict, you just need to type

```sh
git mergetool
```

and git will open p4merge with your conflicts loaded in, ready for you to do some resolving. When you're done with the resolutions,
save the files and commit your changes and your merge is complete.

## dealing with backup files - who needs backups anyway?

Git is pretty careful, so by default it will save the original files that you merged (the ones with the conflict markers in them)
as the filename with a `.orig` extension. You'll either need to delete these or at least add `*.orig` to your `.gitignore` file. You
could also tell git to stop being so darned conservative and turn off the backups with another entry in the `.gitignore` file.

```ini
[mergetool]
keepBackup = false
```

This setting will convince git to cease and desist polluting your directories with `.orig` files. (I would recommend adding the `*.orig`
line to `.gitignore` as well, in case any of your teammates are not taking advantage of the `keepBackup = false` setting)

## setting up the diff tool - same song, second verse

p4merge can also be used to compare files without trying to merge them. You can tell git to use p4merge for diffs by setting up a couple
more entries in the `.gitconfig` file. If you're using Visual Studio Code, you'll probably always use the excellent comparison feature that
is built in on the git tab of the application, but for completeness' sake, let's walk through the steps.

The concept is the same as for the mergetool, except now we're setting up difftool and the command line has fewer options to deal with.
You'll need to add...

```ini
[difftool "p4merge"]
  cmd = C:/Program\\ Files/Perforce/p4merge.exe \"$LOCAL\" \"$REMOTE\"
```

...to declare the difftool and...

```ini
[diff]
  tool = p4merge
```

...to assign p4merge as your diff tool of choice. Now, you just type

```sh
git difftool
```

...on the command line to have git open p4merge for you and send it the files to compare.

## declaring more tools - not a necessary step

Git allows you to declare as many merge or diff tools as you want, then uses the [merge] or [diff] sections to select the name of the tool it uses.
Using this capability, you can set up all of the options from above and switch between them to see which you like best. Rather than step through each
one individually, I'll just paste in the pertinent parts of my `.gitconfig` file.

```ini
[diff]
  tool = p4merge
[merge]
  tool = p4merge
[mergetool]
  keepBackup = false
[difftool "diffmerge"]
  cmd = C:/Program Files/SourceGear/Common/DiffMerge/sgdm.exe \"$LOCAL\" \"$REMOTE\"
[mergetool "diffmerge"]
  cmd = C:/Program\\ Files/SourceGear/Common/DiffMerge/sgdm.exe -merge -result=\"$MERGED\" \"$LOCAL\" \"$BASE\" \"$REMOTE\"
  trustExitCode = true
[difftool "p4merge"]
  cmd = C:/Program\\ Files/Perforce/p4merge.exe \"$LOCAL\" \"$REMOTE\"
[mergetool "p4merge"]
  cmd = C:/Program\\ Files/Perforce/p4merge.exe \"$BASE\" \"$LOCAL\" \"$REMOTE\" \"$MERGED\"
  trustExitCode = false
[difftool "kdiff3"]
  cmd = C:/Program\\ Files/KDiff3/kdiff3.exe
[mergetool "kdiff3"]
  cmd = C:/Program\\ Files/KDiff3/kdiff3.exe
  trustExitCode = false
```

(*If you're wondering why kdiff3 got away without including all those command line parameters, it seems that git is biased, and natively knows
the correct order of parameters that kdiff3 likes.*)

Now that I have all three tools set up, I can switch between them just by changing the value of the `tool` line of the `[merge]` or `[diff]` section.
You can do this either with your editor, or on the command line. To quickly change your merge tool from the command line, you'd type...

```sh
git config --global merge.tool kdiff3
```

...and to set your diff tool to diffmerge, you'd type...

```sh
git config --global diff.tool diffmerge
```

You get the idea.

## git merge success

You should now have at least one external merge tool set up so that you can easily conquer the next merge conflict
that git and your teammates throw at you. Now get out there and build something!
