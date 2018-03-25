shell_command_lock - Atomic locking for shell commands.

**PUBLIC DOMAIN**

https://github.com/jakeogh/shell_command_lock

Prevent identical command lines from executing concurrently. A command line is the combination of the command and it's arguments: $0 $*

Requires: sh, sha1sum

*Theory:*

- 1. Generate unique and reproducible string from $0 $* (the script file name and all it's arguments). sha1sum($0 $*) is used.
- 2. Obtain atomic lock.
- 3. Write $$ (the current PID) to the lockfile.

*Install:*

Place in $PATH.

```sh
$ mkdir ~/bin ; ln -s -r shell_command_lock ~/bin/shell_command_lock
```

*Use:*

insert:
```sh
source shell_command_lock
```
or
```sh
. shell_command_lock #(avoids the 'source' bashism)
```
_before_ the critical section in the parent script. The lock is removed when the parent script terminates via the trap below.

This script should have no effect on the parent script other than locking. The variable names are set to readonly to prevent silent collisions with names in the parent script. The set commands are done in subshells so we don't need to save and restore the state.

*Design notes:*

- This script attempts to be strictly POSIX (no extensions) compliant.
- This script does not depend on bash specific features.
- Redirection using noclobber is the atomic locking primitive instead of mkdir because in it's faster.

*Benchmarks (mkdir vs noclobber):*
``` sh
$ time for x in {1..24000} ; do /bin/mkdir lock ; /bin/rmdir lock ; done
```
```sh
$ time for x in {1..24000} ; do set -o noclobber; :> lock ; /usr/bin/unlink lock ; done
```

*Bugs: (unavoidable?)*

- 1. IMPORTANT: If the trap is re-defined in the parent script, then that trap will need to handle deleting the lock.
- 2. The lockfile is orphaned if a exit signal happens after the lock is obtained and before trap is set.

*More information:*

 - man flock
 - http://www.davidpashley.com/articles/writing-robust-shell-scripts.html
 - http://wiki.bash-hackers.org/howto/mutex
 - http://mywiki.wooledge.org/BashFAQ/045
 - http://wiki.grzegorz.wierzowiecki.pl/code:mutex-in-bash
 - http://code.google.com/p/pylockfile/
 - https://github.com/skx/sysadmin-util/blob/master/with-lock
 - https://github.com/jaysoffian/dotlock
 - http://sysadvent.blogspot.com/2008/12/day-9-lock-file-practices.html
 - http://apenwarr.ca/log/?m=201012#13
 - https://news.ycombinator.com/item?id=2000349

