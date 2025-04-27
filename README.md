# s6-smj

A minimalist init/startup system based on skarnet's [sys-apps/s6][1] and [sys-fs/mdevd][2], offered as an alternative to [sys-apps/s6-linux-init][3].

## Differences from [sys-apps/s6-linux-init][3]

[sys-apps/s6-linux-init][3] uses a unique system involving the generation of a database at configuration time.  By analyzing dependencies, optimal execution order is recorded to the database. (and mutual dependency loops are detected)  This results in ideal boot times, and the fewest running tasks, and least resource consumption.

The downside of this approach is that any minor change to the configuration requires regeneration of the database, and re-initialization.  The complexities this introduced, overwhelmed my cognitive comfort level, driving me to produce s6-smj. `s6-smj` is just another in a long list of things I've done to reduce operating system complexity to a personally auditable level. It is left as an exercise to the reader to deduce whether this is a 'personal problem', or a security measure.

`s6-smj` is a logically simpler approach implemented entirely by execline scripts, and differing largely around how the one-shots (processes run at boot time, but not left running as daemons or services) are handled. Dependencies are not computed at configuration time and stored in a database, but are simply declared by each service or one-shot.  These processes are not launched in optimal order. Instead they block waiting for their dependencies to become available.

Those who use s6-linux-init, may still find these scripts useful within that system.

## Long Running Services

Long running services and logging are managed by `s6` in the usual way. I've opted to put the logs into /run/log to minimize writes to solid state storage, and allow for a read-only root filesystem, at the expense of losing the logs upon reboot or power cycle. Change this behavior by editing the `log/run` file in the service directory.

## One Shots

Scripts that run once when booting, and possibly again when shutting down, are managed by the `1shot` service. Each one-shot is a sub-directory in the `data` directory of the `1shot` service directory.

## Dependencies

Services and one-shots block waiting for long-running services to be ready, by using the `s6-svc -U {servicedir}` command.

Services and one-shots block waiting for one-shots to complete, by calling a script that attemtps to read from a named pipe unique to each one-shot.  When a one-shot completes, it closes the pipe, and renames it, so that tasks reading from it will unblock, and future tasks will find the pipe missing and continue.

Each one-shot logs to it's own log file in it's service directory.

## How To Use

At this time, very little is "automagic".  I like it this way.  I do not choose to run services casually, and want to manage exactly how each one runs.

I've included [execline][4] scripts for the most basic services and one-shots.

There are some quirks unique to how I do things, but they are easy to change.

For instance, I do not use [dbus][5], so I use [eiwd][6] to make wifi connections.  I configure it by editing the text files in `/var/lib/iwd/`, and I connect and disconnect by turning the wifi off and on with keyboard function keys, which prompts `mdevd-as-an-admin` to bring the `iwd` service up and down.

If you choose to use this system, it is likely because you also wish to be able to fully understand what your computer is doing. You are therefore likely to want to read the scripts I have written, understand them, and likely make changes.  This makes that easy to do.

If services have issues, you should see that in their logs. If one shots have issues, you should see the text 'FIXME' in their logs within their service directorys.

When all the one shots are working correctly the `1shot` log file should report that they all completed successfully.

## Status

I am using this system on all my personal computers, and all of my personal servers, and I am very satisfied with the results.  For reference, the servers are mostly ZimaBoards running an "immutable" system (squashfs w/f2fs-overlay), and a few ancient 486 based wifi-routers running the same system.

[1]: https://www.skarnet.org/software/s6/
[2]: https://skarnet.org/software/mdevd/
[3]: https://www.skarnet.org/software/s6-linux-init/
[4]: https://www.skarnet.org/software/execline/
[5]: https://www.freedesktop.org/wiki/Software/dbus/
[6]: https://github.com/illiliti/eiwd
