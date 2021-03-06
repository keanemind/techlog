---
title: Piped Commands Run Concurrently
date: '2018-09-10T17:40:21-05:00'
draft: false
---
As I was learning about Unix piping in preparation to write a shell for my operating systems class, I discovered something new on Stack Overflow that I had never before realized during my limited experience with pipes: Piped commands run concurrently, and could be started in any order.

Let's use an example: `ps | less`
Intuitively, you might think that `ps` runs to completion, and then `less` runs. In reality, you cannot know for sure which program will be started first, and both programs will be running at the same time.

There are two possibilities with the above example:
1. `ps` (process status) is started first, and does whatever it does to determine the currently running processes before `less` is started. Then less runs and starts receiving text through the pipe from `ps`. The list of processes that `less` displays does not have `less` in it.
2. `less` is started first, and waits for input from the pipe. `ps` is then started, and determines the currently running processes. This time, `less` is already running, so `less` is included in the list. Now `less` receives the list through the pipe and displays it.

The reason you cannot know for sure which program will run first is because the shell uses an operating system call, `fork()`, to run the two programs. The order in which the programs are started is determined by the operating system's scheduler. When the shell uses `fork()` to spawn the child processes `ps` and `less`, the processes go into a list of 'ready' processes. The scheduler picks processes to start from that list based on an algorithm, and it is not straightforward to predict which process will be started first. The child processes, once they are both active, are run concurrently. This means that the OS switches between the two processes, who take turns running with each other, as well as with all the other processes currently active on the machine. The program on the right of the pipe character, if it requires input, will wait whenever necessary for the left program to send data through the pipe.

Why does any of this matter? Sometimes, start order of piped programs matters. Let's say you want to read from a file and then overwrite it based on what you read. Something like this: `grep "some search pattern" some_file | tee some_file`

You want `grep` to read from `some_file`, search for "some search pattern", and then have `tee` write the result of the search (the lines the pattern was found in) back to `some_file`.

You have a big problem if `tee` starts first, though. If you provide `tee` a file to write to, the first thing `tee` does when it starts up is clear the file if it already exists. This is so that `tee` overwrites any preexisting data in the file.

By the time `grep` reads the file, it will have been emptied by `tee`! As explained earlier, this problem will not occur every time. Sometimes you'll get lucky and `grep` will be able to do its search before `tee` gets a chance to clear the file. Other times, you won't be so fortunate.

Oh, and by the way, `grep "some search pattern" some_file > some_file` won't work in Bash either, because Bash deals with file redirections first. Bash will clear some_file before executing `grep`.

Alright, that's all I have for you today. Be careful out there!
