+++
title = "Let's talk about exit codes"
[taxonomies]
tags = [ "posix", "unix" ]
+++

Before [starting a shell](https://www.nushell.sh/), if you would have asked me what an exit code was, I would have quickly replied "0 for success, non-zero for failure" without batting an eye. That's what it meant, right?

Today, someone filed an [issue on the Nushell repo](https://github.com/nushell/nushell/issues/4082). In it, they ran `git log`, exited, and Nushell reported the command to have failed.

To know why it happens, we've got to do some digging. First, let's take a look at the standards to understand what we should expect.

# What the standard says

## Bash

One of the first places you'll learn about exit codes is the Bash manual. It says:

"A successful command returns a 0, while an unsuccessful one returns a non-zero value that usually may be interpreted as an error code. Well-behaved UNIX commands, programs, and utilities return a 0 exit code upon successful completion, _though there are some exceptions_." (emph. mine)

## POSIX

The POSIX Open Group Base standard says:

"The value of status may be 0, EXIT_SUCCESS, EXIT_FAILURE, [CX] [Option Start]  or any other value, though only the least significant 8 bits (that is, status & 0377) shall be available from wait() and waitpid(); the full value shall be available from waitid() and in the siginfo_t passed to a signal handler for SIGCHLD. [Option End]"

On the same page it says: "The functionality described on this reference page is aligned with the ISO C standard. Any conflict between the requirements described here and the ISO C standard is unintentional. This volume of POSIX.1-2017 defers to the ISO C standard."

## ISO C

Okay, so what's the ISO C standard say?

"If the value of status is zero or EXIT_SUCCESS, an implementation-defined form of the status successful termination is returned. If the value of status is EXIT_FAILURE, an implementation-defined form of the status unsuccessful termination is returned. Otherwise the status returned is implementation-defined."

"an implementation-defined form of the status unsuccessful termination is returned" feels unsatisfying, but at least we know that non-zero means unsuccessful, right?

# The bug

The original report was about `git log`. Let's run the steps the user reports: run `git log`, quit with `q`, and then check the exit code. I'll do this in bash, since that's the shell everyone will have access to:

```
> git log
> echo $?
141
```

Sure enough, a non-zero exit code.

Bash is reporting that `git log` exits with exit code 141. This clearly feels intentional. It's not a 0 or 1, but a number specific to the other 254 possibilities.

Turns out that [some folks dug into this 5+ years ago](https://www.ingeniousmalarkey.com/2016/07/git-log-exit-code-141.html). They write a bullet list with their findings:

* Git pipes the logs into less
* I quit the less process, which sends a SIGPIPE signal (13) to the underlying git process streaming the logs
* git catches the interrupt and exits prematurely and per POSIX convention returns 128 + the SIGPIPE status ==> 141 to indicate that it was terminated by signal 13. 

# When failures aren't failures

Why is the user being told about the SIGPIPE in this case? I'm sure there's a perfectly reasonable set of events that take us to this point. Git is calling out to `less`, and `less` wants to report the user quitting it ends the pipe prematurely. And yet, it definitely feels to fall short of the _spirit of the specifications_. Non-zero errors *should* mean we've exited in an unexpected way. The steps above exit us in about the only reasonable way to exit interactively viewing the log.

As it turns out, `git log` isn't alone. Here's a discussion on [Stack Overflow about `grep -q` returning 141](https://stackoverflow.com/questions/19120263/why-exit-code-141-with-grep-q). In it, the responses point out yet other commands, like `head`, also return 141 in their regular course of action.

# There are standards, and there are standards

Something folks tend to assume is that the POSIX standard covers a lot of how Unix-like systems work. While this is true at some level, there are cases like this one where we quickly shift to having to lean more on conventions than standards. As the Bash manual says "though there are some exceptions". Using pipes and command composition leads to having to follow one part of the standard that in turn leads to really confusing situations for the user, like the situations listed above. A command like `head` or `less` doesn't take all of its inputs. As it closes, it passes along that "hey, technically the pipe I was reading from isn't going to be able to write to its output because I'm closing". From the user's perspective, though, exiting early is exactly what's supposed to happen. It's not actually an error.

It kinda makes you wonder - if we could do this over again, how would we do it differently?
