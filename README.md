# Punch
<!-- vi: set et ts=2 sw=2 : -->
## What
Yet another commandline time tracking tool.

Yes, they are all named Punch.

See the section at the end of the document for a survey of the others.

This `punch` is distinguished by:

* Its extreme **simplicity**: `punch` draws from a list of tasks (`punch
  tasks`, `punch in`), tracks your current task (`punch now`), and appends it
  to a timesheet file when you clock out (`punch out`, `punch times`).
* Its **human-editable** file format. No XML fake human-friendly broken
  promises here.
* Its ISO 8601 **standards-based** representation of times and durations.
* Its **minimalist** concept of a task: one-word name, start time,
  duration, and a free-form comment in which you can encode whatever other info
  you need or want.
* Its **refusal** to be all things. Small and focused is more malleable than a
  Swiss army knife of tools. Make your data simple enough, and you can slice
  and dice it however you want using a pipeline to other tools.

## Why
Because I failed to do due diligence and use existing tools.

Because I thought, "How hard could this be?"

Because I wanted something whose file format was dead stupid so I could
watch over its shoulder or hand-edit as needed. Check out the head of
`punch-in` for a complete description of the format used in the three files.

Because I wanted something whose output I could readily
feed into [Redminer](https://github.com/jeremy-w/redminer):

    awk '{gsub("-", "/", $1); printf("%s %s %s\n", $1, $2, $3)}' \
        <(punch-summary) | xargs -n 3 redminer time post

## How
Symlink `punch-in` to `punch-out`, `punch-now`, and `punch-summary`.
Then just call the right tool from the commandline.

If you're already punched-in, punch-in will punch-out before punching in.

## Example
```shell
% export PUNCH_DIR=~/.punch-tmp
% punch in
punch: no tasks found -- add some with `punch tasks add`
% punch tasks add task-name default comment for this task\'s time entries
punch: added task: task-name / default comment for this task's time entries
% punch tasks add other:1234 task names are just one word
punch: added task: other:1234 / task names are just one word
% punch tasks add work:101 but feel free to embed an external issue_id
punch: added task: work:101 / but feel free to embed an external issue_id
% punch tasks add client-name/project-name/task-name or entire hierarchy
punch: added task: client-name/project-name/task-name / or entire hierarchy
% punch in
1: task-name (default comment for this task's time entries)
2: other:1234 (task names are just one word)
3: work:101 (but feel free to embed an external issue_id)
4: client-name/project-name/task-name (or entire hierarchy)
punch: enter a number to select a task: 1
punch: punched in: 0m01s task-name (default comment for this task's time entries)
% ls $PUNCH_DIR
current  tasks
% cat "$PUNCH_DIR/current"
task-name 2012-03-14T20:33:29-04:00/2012-03-14T20:33:29-04:00 default comment for this task's time entries
% cat "$PUNCH_DIR/tasks"
task-name / default comment for this task's time entries
other:1234 / task names are just one word
work:101 / but feel free to embed an external issue_id
client-name/project-name/task-name / or entire hierarchy
% punch
punch: current task: 1m29s task-name (default comment for this task's time entries)
% punch out that was fast
punch: punched out: 1m38s task-name (that was fast)
% ls "$PUNCH_DIR"
tasks  times
% cat "$PUNCH_DIR/times"
task-name 2012-03-14T20:33:29-04:00/2012-03-14T20:35:06-04:00 that was fast
#^ 0.03 h = 98 s
% punch times
all times:
2012-03-14 0:01:37 task-name (default comment for this task's time entries)
% punch times --on today --template '{duration|hours/1u} {name} {start|time}-{end|time}'
all times after 2012-03-14T00:00:00-04:00 and before 2012-03-15T00:00:00-04:00:
2012-03-14 0.1 task-name 20:33:29-20:35:06
% punch in 'work*'
punch: punched in: 0m01s work:101 (but feel free to embed an external issue_id)
% punch
punch: current task: 0m05s work:101 (but feel free to embed an external issue_id)
% punch out
punch: punched out: 0m09s work:101 (but feel free to embed an external issue_id)
% punch help
punch: available help topics:

  files -- files and file formats
  glob -- globs used by task list, task remove, times
  template -- template syntax used by times --template option
  tips -- non-obvious uses
  topics -- you're reading it now
  walkthrough -- basic setup and use
  what -- a more complete description

New to punch? Check out walkthrough, concepts, and files.
% unset PUNCH_DIR
```

## The Other Punches
You might want to check the others out:

* [punch](https://github.com/pyrat/punch) (in Ruby)

  - Similar interface.
  - More features.
  - More flexible.
  - Uses subcommands rather than symlink-renaming.
    (This punch is headed that way, too.)
  - Opaque file format (YAML::Store).

* [one_inch_punch](https://github.com/ymendel/one_inch_punch) (in Ruby)

  - Spawn of the just-mentioned punch.
  - Compatible with its file format, so no win there.
  - More flexible.
  - More modular - usable as a library rather than just as a commandline tool.
  - Somewhat newer.

* [punch-clock](https://github.com/dsw/punch-clock) (in Python)

  - Small.
  - Interface uses --too --many --options for my taste.
    Really, `--com=start` to select the command?
  - File format is pretty straightforward:

      - Each command requires you specify a project.
      - You are expected to issue a start command, one or more note
        commands, then a stop command.
      - Each command appends a line to
        `~/punch-clock-db/PROJECT-YYYY-mm`.
        Stop throws in an extra newline to space things out between tasks,
        assuming you have just one in flight at a time.
        So you might see:

            UTC-2012-03-03/20:57:26 start:TPS reports
            UTC-2012-03-03/20:58:26 note:this sucks
            UTC-2012-03-03/21:00:26 stop:quit job

            UTC-2012-03-03/21:01:26 start:drinking till blackout
            UTC-2012-03-03/21:12:26 note:i thibk blkcout s clos nwo

  - A separate `punch-report` tool dumps a summary for a particular
    month in a particular year for a single project.

    - The year/month is inherited from the file format and depends on UTC
      time, which can mess things up around month-end if you're not in a +0
      timezone.
    - The report totals gross given an hourly rate,
      but is basically just a recapitulation of the file with some
      '\tdelta SECONDS' injected after each stop command.
    - It mostly serves to sanity-check the database (did you forget to
      clock out one day, did you clock in without clocking out, etc.).
