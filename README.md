# Committing individual hunks and manually editing hunks

In Git, changes are usually added to the staging area on a per-file base, as `git add foobar.txt` will stage all changes in the file *foobar.txt*. However, with `--patch` (`-p`) `git add` does provide an option for more granular staging.

Let's start working on a file `foobar.txt`:

```
This is the first line.
This is the second line.
This is the third line.
This is the fourth line.



These are some
additional lines way below
the lines at the top
of the file.

```

and add some new lines:

```
This is the first line.
This is a new line between the first and second line.
This is the second line.
This is the third line.
This is the fourth line.
This is a new line after the fourth line.



These are some
additional lines way below
the lines at the top
of the file.
This is a new line at the end of the file.

```

Running `git diff` will produce the follwing output for `foobar.txt`:

```
diff --git a/foobar.txt b/foobar.txt
index 5459acf..fa7b5a5 100644
--- a/foobar.txt
+++ b/foobar.txt
@@ -1,7 +1,9 @@
 This is the first line.
+This is a new line between the first and second line.
 This is the second line.
 This is the third line.
 This is the fourth line.
+This is a new line after the fourth line.



@@ -9,4 +11,5 @@ These are some
 additional lines way below
 the lines at the top
 of the file.
+This is a new line at the end of the file.

```

Now let's assume we only wanted to stage the new line at the end of the file for our commit. Since all changes are in this single file, we'd have to use the `--patch` option for `git add`. Git will then enter an interactive mode and give us the option to stage changes on a per-hunk base, with hunks being individual changesets within a file.

The `git diff` output above shows two hunks, identifiable by their respective `@@ -1,7 +1,9 @@`. and `@@ -9,4 +11,5 @@`.

For each detected hunk, `git add --patch` will (amongst other option) allow us to stage or not stage that hunk:

```
diff --git a/foobar.txt b/foobar.txt
index 5459acf..fa7b5a5 100644
--- a/foobar.txt
+++ b/foobar.txt
@@ -1,7 +1,9 @@
 This is the first line.
+This is a new line between the first and second line.
 This is the second line.
 This is the third line.
 This is the fourth line.
+This is a new line after the fourth line.



Stage this hunk [y,n,q,a,d,/,j,J,g,s,e,?]?
```

and

```
@@ -9,4 +11,5 @@ These are some
 additional lines way below
 the lines at the top
 of the file.
+This is a new line at the end of the file.

Stage this hunk [y,n,q,a,d,/,K,g,e,?]?
```

Choosing `n` (= no) for the first hunk and `y` (= yes) for the
second hunk will result in what we wanted to achieve: Despite all changes being in a single file, only the new line at the end of the file is being added to the staging area, the other other changes remain unstaged. This can be verified by `git status` and `git diff`:

```
On branch master
Changes to be committed:
  (use "git reset HEAD <file>..." to unstage)

	modified:   foobar.txt

Changes not staged for commit:
  (use "git add <file>..." to update what will be committed)
  (use "git checkout -- <file>..." to discard changes in working directory)

	modified:   foobar.txt
```

```
diff --git a/foobar.txt b/foobar.txt
index 73467e2..fa7b5a5 100644
--- a/foobar.txt
+++ b/foobar.txt
@@ -1,7 +1,9 @@
 This is the first line.
+This is a new line between the first and second line.
 This is the second line.
 This is the third line.
 This is the fourth line.
+This is a new line after the fourth line.
```

`git commit` will then commit the staged changes. The result can be verified with `git show`:

```
Date:   Sun Nov 5 18:24:48 2017 +0100

    Add a new line at the end of the file.

diff --git a/foobar.txt b/foobar.txt
index 5459acf..73467e2 100644
--- a/foobar.txt
+++ b/foobar.txt
@@ -9,4 +9,5 @@ These are some
 additional lines way below
 the lines at the top
 of the file.
+This is a new line at the end of the file.
```

But what if we now wanted to only commit the new line between the first and the second line in the next commit? Git interpretes the changes as a single hunk only, so `git add --patch` will only present us that single hunk in the interactive staging:

```
diff --git a/foobar.txt b/foobar.txt
index 73467e2..fa7b5a5 100644
--- a/foobar.txt
+++ b/foobar.txt
@@ -1,7 +1,9 @@
 This is the first line.
+This is a new line between the first and second line.
 This is the second line.
 This is the third line.
 This is the fourth line.
+This is a new line after the fourth line.



Stage this hunk [y,n,q,a,d,/,s,e,?]?
```

In this case, we need to manually edit the hunk for staging with the option `e`, which will open the respective hunk in a text editor:

```
# Manual hunk edit mode -- see bottom for a quick guide.
@@ -1,7 +1,9 @@
This is the first line.
+This is a new line between the first and second line.
This is the second line.
This is the third line.
This is the fourth line.
+This is a new line after the fourth line.



# ---
# To remove '-' lines, make them ' ' lines (context).
# To remove '+' lines, delete them.
# Lines starting with # will be removed.
#
# If the patch applies cleanly, the edited hunk will immediately be
# marked for staging.
# If it does not apply cleanly, you will be given an opportunity to
# edit again.  If all lines of the hunk are removed, then the edit is
# aborted and the hunk is left unchanged.
```

To successfully edit this hunk, we need to understand what `@@ -1,7 +1,9 @@` actually means: These two pairs of numbers explain the context of this hunk before (`-`) and after (`+`) the changes. For this specific example `-1,7` means the file content displayed for this hunk before the changes starts at line 1 of the file, and shows 7 lines (including the first line, and not counting the lines that make up the changeset, which in this case are the two lines starting with `+`), while `+1,9` means, for the context after the changes, the content displayed starts at line 1 of the file as well, and shows 9 lines (that's two more because we added two more lines).

To achieve the result we want (only add the line between the first and the second line to the staging area), we need to not only remove the undesired line `+This is a new line after the fourth line.`, but also adjust the numbers describing the context to make the patch apply cleanly (otherwise it will fail).

Modifying the text in the editor to the following:

```
@@ -1,2 +1,3 @@
This is the first line.
+This is a new line between the first and second line.
This is the second line.
```

tells Git to treat this hunk as starting on the first line of the file, showing two lines pre-changes, and starting at the first line of the file, showing three lines post-changes. Saving the file and exiting the text editor will apply the patch and stage the hunk if it does apply cleanly. This can be verified with `git status` and `git diff`:

```
On branch master
Changes to be committed:
  (use "git reset HEAD <file>..." to unstage)

	modified:   foobar.txt

Changes not staged for commit:
  (use "git add <file>..." to update what will be committed)
  (use "git checkout -- <file>..." to discard changes in working directory)

	modified:   foobar.txt
```

```
diff --git a/foobar.txt b/foobar.txt
index 105035e..fa7b5a5 100644
--- a/foobar.txt
+++ b/foobar.txt
@@ -3,6 +3,7 @@ This is a new line between the first and second line.
 This is the second line.
 This is the third line.
 This is the fourth line.
+This is a new line after the fourth line.


```

This confirms at least one hunk of `foobar.txt` has been added to the staging area, and unstaged changes in `foobar.txt` only include the new line after the fourth line.

Committing the changes and then running `git show` shows the desired result:

```
Date:   Sun Nov 5 23:23:55 2017 +0100

    Add a new line between the first and second line.

diff --git a/foobar.txt b/foobar.txt
index 73467e2..105035e 100644
--- a/foobar.txt
+++ b/foobar.txt
@@ -1,4 +1,5 @@
 This is the first line.
+This is a new line between the first and second line.
 This is the second line.
 This is the third line.
 This is the fourth line.
```
