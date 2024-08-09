---
title: "Bash Scripting - Mass VM SSH Connections for Fun and Giggles"
date: 2023-11-14T11:27:08-08:00
tags: ["bash", "shell", "script", "SSH"]
toc: true
description: "Is it illegal if it's fun, harmless, and educational? (Don't answer that)"
draft: false
---

## Introduction

I am currently finishing my last semester of university. My final
computer science elective I am taking is Linux Administration.
It is a fairly easy and straightforward class, but the reason I am taking
this class is to learn more about the Linux Operating System, specifically
when it comes to system administration.

One of the topics I wanted to learn about is bash scripting. I have seen many
bash scripts online, but I have never taken the time to learn how to read or
write them. We did have two lectures about bash scripting with their own lab,
which is great, but I wanted more experience with bash scripts.

Halfway through the semester, we transition into the virtual machine portion
of the class, where we get to be sysadmins of our own virtual Linux
distributions. Most of the students, including me, use the school computers,
which are connected together in the same network. You can also SSH from the
school computer into your virtual machine if you set the connection type to
bridge. Because of this, I can SSH not only into my virtual machine, *but also
other students' machines*. A mischevious idea brewed in my head that night.

After one long coding session late at night, and a couple rewrites and
modifications, I finally had an automated shell script that will SSH into
**every** students' virtual machine and place a text file on their desktop --
a signature from me proclaiming the time and date I "hacked" into their system.

## The Bash Script

Here is the bash script:

```bash
#!/bin/bash

# The purpose of this script is to ssh into VMs on the network and place a harmless text file on their desktop

OS=$2

show_help() {
        echo "Usage: sshtxtmsg [file] [operating-system]"
        echo "  File must be formated: Last,First"
        echo "  Must have 'sshpass' installed!"
}

ssh() {
        LNAME=`echo $1 | egrep -o '^[[:alpha:]]*'`
        #echo "$LNAME"
        FNAME=`echo $1 | egrep -o '[[:alpha:]]*$'`
        #echo "$FNAME"
        THREEF=`echo $FNAME | egrep -o '^[[:alpha:]]{1,3}'`
        #echo "$THREEF"
        THREEL=`echo $LNAME | egrep -o '^[[:alpha:]]{1,3}'`
        #echo "$THREEL"
        HNAME="${OS}-${THREEF}${THREEL}"
        #echo "$HNAME"
        USERNAME=`echo $FNAME | tr [:upper:] [:lower:]`
        #echo "$USERNAME"
        COMMAND="${USERNAME}@${HNAME}"
        #CMD='sshpass -p MP2650 ssh -o StrictHostKeyChecking=no -q -l root $HNAME "echo "EVNMRK WAS HERE ON " > ~/Desktop/EVNMRK-WAS-HERE.txt; date >> ~/Desktop/EVNMRK-WAS-HERE.txt"'
        CMD='sshpass -p cmps2650 ssh -o StrictHostKeyChecking=no -q -l $USERNAME $HNAME "echo "EVNMRK WAS HERE ON " > ~/Desktop/EVNMRK-WAS-HERE.txt; date >> ~/Desktop/EVNMRK-WAS-HERE.txt"'
        #CMD1="echo '$COMMAND'"
        eval "$CMD"
        if [ $? -eq 0 ]; then
                echo "SUCCESS for $COMMAND - $1"
        elif [ $? -eq 1 ]; then
                echo "FAILURE for $COMMAND"
        else
                echo "UNKNOWN ERROR"
        fi
}

# MAIN

if [ $# -lt 2 ]; then
        show_help
        exit 1
fi
for name in `cat $1`; do
        ssh "$name"
done
```

### Usage

```
Usage: sshtxtmsg [file] [operating-system]
	File must be formated: Last,First
	Must have 'sshpass' installed!
```

### How It Works

The script requires a text file with every students' name in the format of
"Last,First" as the first argument. Each name is separated by a newline.
The script also requires the name of the operating system you are connecting
to as the second argument. The reason for this is because of how we set up the
virtual machines as a class. The instructor had us follow instructions on how
to set up our virtual machines. Part of this instructions is what to put for
the hostname, username, password, and root password. These fields are
determined by the students' first and last name, which is why we need the
student roster. The hostname field has the operating system's name inside, too.

- Example Debian hostname for student "Dale Cooper"
	- `debian-dalcoo`
- Example username
	- `dale`
- Example ssh command
	- `ssh dale@debian-dalcoo`

Using regex, we can splice the first and last name to create a username and
hostname. But we run into a problem: `ssh` cannot be automated, since it
queries for a password. To solve this, we use the very unsecure `sshpass`,
which can hold the password in a flag. The password of the user is the same for
everyone: the classname. We also add the `StrictHostKeyChecking=no` flag to
ignore asking to remember the ssh fingerprint for 100% automation. Here is
that command:

`sshpass -p cmps2650 ssh -o StrictHostKeyChecking=no -q -l $USERNAME $HNAME`

We can append the commands to run once a connection is made to the end of the
sshpass command. We just run an echo command for our text and redirect it into
a new textfile. Then we append the datetime the same way.

`"EVNMRK WAS HERE ON " > ~/Desktop/EVNMRK-WAS-HERE.txt; date >> ~/Desktop/EVNMRK-WAS-HERE.txt"`

The script will do the previous commands for every student entry, outputting 
a success or failure.

### Known Issues

This script will not work for users who choose their own unique hostname,
username, or password. It will also not work for users who install their
virtual machine on their own laptop. Basically it doesn't work on students
who are lazy, cannot RTFM, or are mavericks.

## Conclusion

Out of the many times I ran this script, I only had three successful
connections.

I also have a version of this script that will run in an infinite loop,
but let's not get into any trouble with that...

Use this script however you like, feel free to modify for your own use.
