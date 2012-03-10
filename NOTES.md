<!-- vi: set et ts=4 sw=4 -->
## Desired Interface
name-glob below means a name optionally including '*' for any chars and
'?' for a single char. '\' is an escape character. (used fnmatch)

    punch tasks list
    punch tasks add/new name and the rest is comments
    punch tasks remove/del/rm -f/--force name-glob name-glob name-glob

`--force` prevents asking for confirmation prior to deletion.

**???: Any way to add command aliases when using argparse? Not until a later
Python -- see [aliases for positional arguments (subparsers)][aliases].

  [aliases]: http://bugs.python.org/issue9234

Or set a default subcommand when none is provided?
Yes, scan the args yourself and stick in the default. Yuck. **

    punch in name-glob
list tasks matching, or if just 1, declare it; no task lists all to choose from

    punch out comment

    punch times

What should punch times do, anyway? How about easy summary/selective listing?

    punch times --after DATE --before DATE --on DATE
    punch times --template 'name: {name}'

Template tokens: see `punch help template`.

## New Arch
* Task -- name, start, duration, comment

    ^(\S+)\s*([^/]+)/(\S+)\s*(.+)$

* TaskStream -- skips comment and blank lines
    - read(fileobj)
    - write(fileobj, taskseq)

* ISO -- handles ISO formatting, UTC->local conversions for summary
    - now() -- current date-time in UTC
    - duration(start=, stop=, duration=) supply any two, it returns the third
    - local(datetime) -- converts to local time

* Path -- global again -- config but named better

* Punch -- the tool, as an object

## TODO
* Pipe help output through a pager when it exceeds a certain number of lines,
  say 18.
* Test the punch times date functionality.
* Add lenient date parser for punch times.
* Explain why we represent the duration as local-start/local-end rather than
  using UTC or start/duration in the format help. (Short version: we can't
  reconstruct the local start/end times from any other representation, but
  local->UTC is easy, and date-time - date-time is also easy.)

     * Actually, this is already in `punch help what`, but probably bears
       repeating.
* Add Help.license()
* Add --comment commentglob to tasks list, tasks remove, times.
    - If both name and commentglob are provided, AND the two lists.