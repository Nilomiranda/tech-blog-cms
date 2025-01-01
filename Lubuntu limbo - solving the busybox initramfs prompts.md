After succesfully setting up a Lubuntu system in an old MacbookAir 2015
and going through the amazing experience of ~~figuring out and installing drivers~~
I was ready to boot it up next day and do some work.

Next day comes and boom, something like this shows up:

```bash
BusyBox v1.18.5 (Ubuntu 1:1.18.5-1ubuntu4) built-in shell (ash)
Enter 'help' for a list of built-in commands.

(initramfs)
```

Thankfully, this was super easy and quick to fix, not something common in the Linux
experience.

I'm going to credit this [link](https://askubuntu.com/questions/137655/boot-drops-to-a-initramfs-prompts-busybox) and more specifically [this answer](https://askubuntu.com/a/817660) for the solution.

I just had to attempt to `exit` first.

I'd see something like this:

```bash

/dev/mapper/ubuntu--vg-root: UNEXPECTED INCONSISTENCY; RUN fsck MANUALLY.
(i.e., without -a or -p options) 
fsck exited with status code 4. 
The root filesystem on /dev/mapper/ubuntu--vg-root requires a manual fsck.
```

Just ran:

```bash
fsck /dev/mapper/ubuntu--vg-root -y
```

And then `exit` again. `reboot` command didn't work, nothing happened, no error
message, nothing. 

Again, all credit to the answer I linked above.
