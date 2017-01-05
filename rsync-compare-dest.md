---
title: Rsync compare-dest option
author: Andrew
date: 2015-12-29
version: 0.0.0
---

Use the `--compare-dest` option to supply a third directory in the rsync
command. Files in SRC are copied to DEST _only if_ they have changed from
the COMPARE directory.

This can be useful to stash the changes between an old snapshot of a directory
and the current state of that directory. For example:

    $ rsync -av \
      --compare-dest="/$CURRENT" \
      "/$SNAPSHOT/" \
      "/$DESTINATION"

Note: This seems backwards, because `$SNAPSHOT` is the SRC argument to rsync,
and `$CURRENT` is the comparision argument. However, this is so that the _old_
versions of files are copied to `$DESTINATION`. Otherwise, you'd be copying the
new version of any files that have changed since the snapshot was taken. Here,
`$DESTINATION` will contain everything from `$SNAPSHOT` that has been changed
in `$CURRENT`.
