# sqldump-manager

Simple bash script to manage and perform SQL backups into plain text .sql files using the built-in utility mysqldump (or mariadb-dump for mariadb). It also works with databases inside docker containers.
The script natively supports mysql, mariadb and postgres databases, but theoretically any other database could be implemented by adding a simple template.

### Config
Databases are to be defined in a conf file (sample provided) in which you can define how to access the database and how to perform the backup.
```
[DEFAULT]
is_template=1
dump_dir=$HOME/sql_backups
dump_bin=/usr/bin/mysqldump
last_sql=copy
delete_older_than_x_days=90
permissions=600

[my_app]
dump_bin=/usr/bin/mariadb-dump
last_sql=symlink
docker_container=my_app_db
db_name=app
db_user=root
db_password=1234

[my_other_app]
...
```
Each [section] represents a database you want to make backups of, you can define as many of this as you want. The first section (default) is for defining global values that will be shared across all sections unless the value is overrided.
Additionally, any section can be defined as a template to set common values by using `is_template=1`. You can then inherit this values in any other section by calling `parent_template=<template name>`. Only one parent is allowed by section.


| Variable | Default value | Description |
|-|-|-|
| `dump_dir` | $HOME/sql_backups | Directory where backups will be saved, a subdirectory will be created for each section |
| `dump_bin` | /usr/bin/mysqldump | Binary used to dump the database, in mariadb containers this might be `/usr/bin/mariadb-dump` |
| `last_sql` | copy | Optionally, after creating the backup a copy or symlink of the generated dump will be created in the same directory. This is for quick referencing. Possible values are: `copy` and `symlink` |
| `docker_container`| | Docker container in which the database resides (optional). |
| `db_name` | | Name of database |
| `db_user` | | User for database |
| `db_password` | | Password for database |
| `delete_older_than_x_days` | 0 | If specified (greater than 0), backups older than this amount of days will be deleted after the backup is completed. |
| `permissions` | 600 | Permissions to give to the generated .sql files |
| `is_template` | 0 | Make this section a template |
| `parent_template` | | Inherit values from a given template |

### Executing
The scripts must be called with one of two arguments
| Command | Effect |
|-|-|
| `sqldump-manager -a` | Create backups for all of the defined sections |
| `sqldump-manager -d <name>` | Create backup for the specified section |
| `sqldump-manager -v` | Print script version |

I wanted this to be as simple and modular as possible so there is no built-in scheduling of backups or nothing like that. The easiest way of automating backups is calling this script from a crontab.
