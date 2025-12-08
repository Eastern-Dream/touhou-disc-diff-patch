This repository stores the diff for Touhou game update. ZUN's executable touhou game patch are notoriously hard to run and use, due to needing Japanese system locale and sometimes directory picker dialog in Japanese that is difficult to understand. I make the diff and instructions redistributable it for anybody to easily patch their copy of Touhou games.

# Patching Prerequisite
### Linux
You only need `git` to provide git apply. Alternatively if you are not using git apply, you need GNU patch from `patchutils`. GNU patch may also exists in different common UNIX utilities program like `busybox` or `toybox`.

### Windows
You need to install Git for Windows (https://git-scm.com/downloads/win). This gives two things that is needed:
- `git apply` and optionally GNU patch
- Bash shell

All instructions can only be done in the Git Bash terminal.

# Patch Procedure
Only applies if you have the original JP game directory, pre-patched one may not work! Unless specified otherwise within the README of each patch folder.

### Using patch script (recommended)
There is an interactive bash script `patch.sh` for the patch procedure. Simply make the script executable and run it in the terminal and follow instructions.

### Using git apply (recommended)
If you are not using the script, please make sure you see relevant section about renaming file extension to lowercase below and do that to your game directory before proceeding. Another procedure is using `git apply`, note that you need to be in the game directory before running command:
```sh
git apply /path/to/patch_to_apply.diff
```
You can also do a glob pattern to a directory containing diff files. Thanks to naming convention, this should always apply patch in the correct order.
```sh
git apply dir_containing_diff_files/*.diff
```

### Using GNU patch
If you are not using the script, please make sure you see relevant section about renaming file extension to lowercase below and do that to your game directory before proceeding. Another alternative procedure not using `git apply` but with GNU patch instead.
Open `readme.txt` to see what game version you currently have and determine what diff to use to patch your game to the latest version. To apply patch, simply dry-run:
```sh
patch -Np1 --no-backup-if-mismatch -d dir_to_apply_patch_onto/ --dry-run < patch_to_apply.diff
```

Windows user need to add `--binary` flag like so to ignore different line endings:
```sh
patch -Np1 --no-backup-if-mismatch --binary -d dir_to_apply_patch_onto/ --dry-run < patch_to_apply.diff
```

When the set of flags above is used, it will automatically skip invalid or previously applied patch file, only leaving reject files. Thus, it is safe to use this command even on wrong diff without dry run. If the patch applies cleanly, remove the `--dry-run` flag and re-run the command.

In case there were bad patches applied and it left a bunch of reject files, simply run this command to clean up:
```sh
find directory_with_bad_patch/ -name '*.rej' -exec rm {} \;
```

# Diff Procedure
You need GNU diff from `diffutils` or `busybox` or `toybox`. Git diff is not supported as it does not produce interoperable patch file. Please only perform this on Linux.

Starts with the original ver 1.00 directory for each game (or the oldest patchable version that is available).

1. Recursively rename all file extension to lowercase (See relevant section below)
2. Create a copy of the game directory to be patched
3. Drop ZUN executable patch into game directory
4. Run ZUN executable patch to completion through WINE 32-bit (with `LC_ALL=ja_JP.UTF-8`, you may also need system JP locale installed)
5. Remove ZUN executable patch from the game directory
6. Generate diff via `diff -Naur before-patch/ after-patch/ > patch_name.diff`

This process is repeated between each patch stage until the latest version is reached. Contributors should follow the same procedure.

The diff file should be named after the name of the ZUN executable patch minus the `.exe` extension. For example, changes made by running `kouma_update102f.EXE` should make the resulting diff be named `kouma_update102f.diff`. Normalization of diff file naming like this allows for search or listing of diff files to yield a correct order to apply patches.


## Recursively rename file extension to lowercase
Sometimes, the file extension in the game directory is a mix of uppercase and lowercase. This behavior is first observed in th06 EoSD disc, where the on-disc directory has uppercase .DAT filename extension. But if the game was installed by running `install.exe` instead of just copying the game directory, then the .DAT filename extension were lowercase to .dat instead. Since diff/patch and linux are case-sensitive in most context, this requires normalizing all file extension to lowercase.

Below is the command snippet that does this in the current directory. Beware that this is extremely destructive if used elsewhere e.g. root.
```sh
find . -name '*.*' -exec sh -c '
  a=$(echo "$0" | sed -r "s/([^.]*)\$/\L\1/");
  [ "$a" != "$0" ] && mv "$0" "$a" ' {} \;
```
