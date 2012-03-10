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
% export PUNCH_DIR="$HOME/.punch-tmp"
% punch-in   
punch-in: no tasks found
% cat >~/.punch-tmp/tasks
SomeTask 1234 whatever comment you want - maybe the project name
Task2 112 The number is an issue_id. Steal it from your issue tracker.
DoSomething 42 Death March
% punch-in
select a task:
0: SomeTask (whatever comment you want - maybe the project name)
1: Task2 (The number is an issue_id. Steal it from your issue tracker.)
2: DoSomething (Death March)
2
% ls ~/.punch-tmp
current  tasks
% cat ~/.punch-tmp/current
42 1330810079
% punch-now
punch-now: DoSomething/42 (Death March) - 0:00:31
% punch-in
select a task:
0: SomeTask (whatever comment you want - maybe the project name)
1: Task2 (The number is an issue_id. Steal it from your issue tracker.)
2: DoSomething (Death March)
1
punch-out: DoSomething/42 (Death March) - 0:00:47
% punch-now
punch-now: Task2/112 (The number is an issue_id. Steal it from your issue tracker.) - 0:00:08
% cat ~/.punch-tmp/current 
112 1330810121
% punch-out
punch-out: Task2/112 (The number is an issue_id. Steal it from your issue tracker.) - 0:00:15
% punch-now
punch-now: no current task
% ls ~/.punch-tmp
tasks      timesheet
% cat ~/.punch-tmp/timesheet
# DoSomething/42 (Death March)
42 1330810079 1330810126

# Task2/112 (The number is an issue_id. Steal it from your issue tracker.)
112 1330810121 1330810136

% punch-summary
2012-03-03 0.5 112 Task2 (The number is an issue_id. Steal it from your issue tracker.)
2012-03-03 0.5 42 DoSomething (Death March)
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
