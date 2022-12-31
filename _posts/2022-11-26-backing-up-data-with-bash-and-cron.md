---
layout: post
title: Backing Up Data With Bash And Cron
author: Carlos Molero Mata
description: Writing a bash script and running it regularly with cron to back up data.
header-style: text
tags: [bash scripting, unix]
---

This time I bring you a super short and simple tutorial to write a bash script that makes a backup of a folder of our choice on a regular basis thanks to the use of cron jobs.

Shall we start?

## Why it is important to backup data regularly?

Data redundancy is one of the most widely used strategies to prevent the loss of valuable information. **Data redundancy refers to the replication of data in different storages** so that if there is a problem in any of them we can recover an updated copy.

In my case, I work with an external hard drive where I have everything I need to code: applications, versions of some dependencies and libraries, copies of client projects etc. I do this because it allows me to have at hand a mini development environment that outlives the computers I use. However, in my current Linux distribution (Linux Mint) the filesystem is somewhat unpredictable and sometimes, because of mount point problems, I have lost information. Therefore, **since a few years ago I always create a cron job that runs a script that daily creates a copy of some key folders of my SSD** on the system where I work.

Read on to find out how I do it!

## The Gist

As you can see, if you have worked with `UNIX` systems in the past, there is nothing too crazy about this script. But let's explain it below anyway.

```bash
#!/bin/bash

DATE=$(date +%d-%m-%Y)
BACKUP_DIR="/backup"
BACKUP_TARGET=$1
BACKUP_NAME=$2

# Zip and save a copy of the source folder to the backup dir
tar \
--exclude=github/**/.git/* \
--exclude=github/**/node_modules/* \
--exclude=github/**/target/* \
--exclude=github/**/.jekyll-cache/* \
--exclude=github/**/build/* \
--exclude=github/**/vendor/* \
--exclude=github/**/.next/* \
-zcvpf $BACKUP_DIR/$BACKUP_NAME-$DATE.tar.gz $BACKUP_TARGET

# Delete files older than 10 days
find $BACKUP_DIR/* -mtime +10 -exec rm {} \;
```

### Variables

- `$DATE` - the [`date`](https://www.geeksforgeeks.org/date-command-linux-examples/) on which the script is executed
- `$BACKUP_DIR` - the folder where we are going to store our backup files
- `$BACKUP_TARGET` - first input (source folder)
- `$BACKUP_NAME` - second input (backup files name)

### Using [`tar`](https://www.computerhope.com/unix/utar.htm) for compression

On Unix-like operating systems, the tar command creates, maintains, modifies, and extracts files that are archived in the tar format.

The flag `-zcvpf` indicates to the command that we want to:

- `z` - use gzip compression
- `c` - create a new file
- `v` - show a detailed output of the operation
- `f` - define a name for the new file
- `p` - do not alter source permissions

The `--exclude` flag, as you probably guessed, is for ignoring some paths inside our source folder.

### Using [`find`](https://www.ionos.com/digitalguide/server/configuration/linux-find-command/) and [`mtime`](https://www.computerhope.com/unix/utar.htm) to delete backups older than N days

In the last line of the script we use `find` and `-mtime` to search for copies older than 10 days and delete them.

`-mtime n` will evaluate as true if the file modification time subtracted from the initialization time, divided by 86400 (with any remainder discarded), is n.

> These last two steps are intended to reduce the weight of the backup files so that we do not run out of storage space on the computer on which we save them.

### Creating the cron job

First, save this file to `/bin/backup.sh`.

Then, open the terminal and type `crontab -e`. Scroll down and add a new line at the end of the file:

`0 11 * * * /bin/backup.sh <BACKUP_TARGET_FOLDER> <BACKUP_FILES_NAME>`

This cron job will execute the script everyday at 11:00 AM. You can create your own cron interval with [CronGuru](https://crontab.guru/) if you're not familiar with the syntax.
