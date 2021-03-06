# Borg cron helper scripts

**Automate backups with borg in a more convenient and reliable way!**
These scripts are some small and handy shell scripts to automate the backup process with [BorgBackup](https://borgbackup.readthedocs.io/). They are POSIX-compatible, so they should run with all shells. You're free to modify them for your needs!

They add some convienent features around borg, regarding environments with only **one client**.

## Features
* **[Local lock system](#local-lock):** Cirumvent the issue of [stale](https://github.com/borgbackup/borg/issues/813) [lock files](https://github.com/borgbackup/borg/issues/2306).
* **[Automated retries](#less-maintenance-more-safety):** When backups [stop mid-way](https://borgbackup.readthedocs.io/en/stable/faq.html#if-a-backup-stops-mid-way-does-the-already-backed-up-data-stay-there) they are automatically restarted.
* **[Simple configuration](config/example-backup.sh):** Using shell-scripts you can configure each backup and then execute it the order/way you want.
* **[Status information](#less-maintenance-more-safety):** You can use a login script to get a notice when backups failed.
* **[Optional & adjustable](#modular-approach):** You do not have to use all features and you can adjust them in a simply way.

### Local lock (borg < v1.1.0)

When the backup process is interrupted, sometimes the remote borg repository stays locked. That's why further backups will fail.

The issue [has been addressed](https://github.com/borgbackup/borg/pull/1674) in borg **v1.1.0** (currently beta), but I have not tested it and until it works there, here is my workaround.

Borg's current PID is written into a file. As long as the file exists the backup is considered to be "locked". At the end of the backup process (no matter whether it was succesful or not), the local lock is being removed, permitting further backups to start. The "local lock" is more reliable than the "remote lock" system, currently implemented in stable versions of borg.

### Less maintenance, more safety!

Sometimes [backups stop mid-way](https://borgbackup.readthedocs.io/en/stable/faq.html#if-a-backup-stops-mid-way-does-the-already-backed-up-data-stay-there). This can have different reasons (e.g. an unreliable network). However we still want our data to be backed up.
That's why the script integrates a **retry mechanism** to retry the backup in case of a failure (for up to three times by default). Between the retry attempts the script pauses some minutes by default, to wait for your server to restart or the connection to reestablish.
This is also a workaround for the ["connection closed by remote" issue](https://github.com/borgbackup/borg/issues/636), which seems to affect some users.

Additionally the script has the ability to write stats of the last executed backup. [`cronsizecache.sh`](cronsizecache.sh) outputs the size of backups stored on the backup server. Both can be used (in conjunction with a script to check the last backup time) to **monitor your backups** and to automatically notify you, in case a backup failed/didn't work.

**Rest assured your backups are safe and recover themselve if possible.** (If they don't, you'll get to know that.)

### Modular approach

The main "work" is done by [`borgcron.sh`](borgcron.sh). This script can be used to execute a single backup.
The configuration files per repository/backup are saved in the  [`config`](config/) directory.
The file [`borgcron_starter.sh`](borgcron_starter.sh) is the file you should call from cron or when debugging manually. Using it, you can execute multiple backups by passing the config names to it.

Also, you can of course not use some features outlined here. That's why the whole functionality is broken into multiple scripts.

### More features…
* easy to understand, easy to modify
* POSIX-compatible
* pretty logs
* passphrase can be protected better, because it's saved in an external file
* pruning after the backup
* script to dump databases before backing up
* privilege-separation (login scripts can have higher privilege than backup process)
* tested in production (but no guarantees, use at your own risk! 😉)
* logging if backup is interruped by signals (e.g. at shutdown)
* more to come…

## What's in here?
* [`borgcron_starter.sh`](borgcron_starter.sh) – Cycles through backup files and interprets passed parameters.
* [`borgcron.sh`](borgcron.sh) Main script. Actually executes the backup and runs borg.
* [`config`](config/) – Directory for config files
   * [`example-backup.sh`](config/example-backup.sh) – Example configuration file for a backup. Please use this template to add your backup(s).
* [`tools`](tools/) – additional scripts
   * [`checklastbackup.sh`](tools/checklastbackup.sh) – Script, which you can execute at login (add it to your `.bashrc` file or so). It notifies you when a backup has failed. Otherwise it remains silent.
   * [`cronsizecache.sh`](tools/cronsizecache.sh) – Small one-liner to cache the size of the dir where backups are stored. (useful for remote backup servers) You can then include the result with `cat` in your login script.
   * [`databasedump.sh`](tools/databasedump.sh) – Dumps one or several databases into a dir/file. Make sure, that this script and the dump dir are only readable by your backup user. Script might have to be executed with higher privileges (i.e. root) for creating the backup.
* [`system`](system/) – Various system scripts, you may need for your setup.

## How to setup?

Setup instructions can be found [in the wiki](https://github.com/rugk/borg-cron-helper/wiki/How-to-setup%3F).
