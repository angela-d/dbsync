# Database Sync Automation

This lightweight script will pull databases from a bzipped backup and import them into your local environment.

Useful for local development environments where you have multiple databases from the same server backed up into a single .sql backup.

## Local Environment
This script makes a few assumptions.

- Linux-based operating system (if not, modify `service mysql start` and stop to suit your OS) running MySQL/MariaDB
- Your databases already exist locally (or in a mounted/otherwise accessible drive)
- The database is compressed in **bz2** format
- The database filenames are in the following context: **sqlbackup-YYYMMDD.sql.bz2**
- Multiple databases are stored in a single backup (ie. from the same server) -- it should work with a sole database, but only tested in situations with numerous databases.
- dbsync is run *after* today's backup is ran (it will look for sqlbackup-20190622.sql.bz2 if today is June 22)

For remote environments, [example crons](https://github.com/angela-d/brain-dump/blob/master/sysadmin/crons/crontab) to generate bzip2 backups can be found in my notes repo.

## Caveats
If your particular environment differs, you'll want to customize the script.

*All* databases are wiped and imported once this script is run, except database names matching:

- Database
- information_schema
- mysql
- performance_schema
- phpmyadmin

If you have database(s) you don't want dbsync touching, modify the following:
```bash
mysql -u"$DEST_DB_USR" -p"$DEST_DB_PWD" -e "show databases" | grep -v Database | grep -v information_schema | grep -v mysql | grep -v performance_schema | grep -v phpmyadmin | gawk '{print "drop database `" $1 "`;"}' | mysql -u"$DEST_DB_USR" -p"$DEST_DB_PWD"
```

Append your entry, like so:
> .. grep -v Database | grep -v information_schema | grep -v mysql | grep -v performance_schema | grep -v phpmyadmin | grep -v ignore_this_database | gawk ..
>

With **ignore_this_database** being the database name of the database you want to remain untouched.

### Limits
The largest (total) .sql file imported with this script was around 3MB.  It could undoubtedly handle much larger, but it's not optimized for it - use at your own risk.

### Synchronization Desktop Prompts (optional)
If you're on a Linux desktop (and have something like [libnotify-bin](https://packages.debian.org/search?keywords=libnotify-bin) installed) you can display prompts when synchronization-related events occur.

Example:

![Desktop notification](rotate.png)
```bash
10 14 * * 0 cd /storage/db_backups/ && /usr/bin/find /storage/db_backups/* -name "sqlbackup-*.sql.bz2" -mtime +30 -exec rm {} \; && notify-send -i access "Synchronization" -u critical "Database rotation complete"
```
This cron runs at 2:10pm on Sunday and deletes any backup older than 30 days.  It will notify me on my desktop once the process is complete.

The above is not part of this script but can be used as a companion cron - modify to suit your environment.

## Install
```bash
git clone https://github.com/angela-d/dbsync.git && cd dbsync && ./dbsync
```
