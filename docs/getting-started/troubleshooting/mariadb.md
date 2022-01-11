# Troubleshooting MariaDB Problems

!!! info ""
    You are welcome to ask for help in our [community chat](https://gitter.im/browseyourlife/community).
    [Sponsors](../../funding.md) receive direct [technical support](https://photoprism.app/contact) via email.
    Before reporting a bug, try to determine the cause of your problem.

!!! note ""
    Official support for MySQL is discontinued as Oracle seems to have stopped shipping [new features and improvements](https://github.com/photoprism/photoprism/issues/1764).
    As a result, the testing effort required before each release is no longer feasible.

#### Unicode Support ####

If the logs show "incorrect string value" database errors and you are running a custom MariaDB or MySQL
server that is not based on our [default configuration](https://dl.photoprism.app/docker/docker-compose.yml):

- [ ] Full [Unicode](https://home.unicode.org/basic-info/faq/) support [must be enabled](https://mariadb.com/kb/en/setting-character-sets-and-collations/#example-changing-the-default-character-set-to-utf-8), e.g. using the `mysqld` command parameters `--character-set-server=utf8mb4` and `--collation-server=utf8mb4_unicode_ci`
- [ ] Before reporting a bug, verify that the problem still occurs with a newly created database based on our example

Note that your database may also use a different character set if you imported it from another server.
Run this command in a terminal to see the current values of the collation and character set variables (change the root
password `insecure` and database name `photoprism` as specified in your `docker-compose.yml`):

```bash
echo "SHOW VARIABLES WHERE Variable_name LIKE 'character\_set\_%' OR Variable_name LIKE 'collation%';" | \
docker-compose exec -T mariadb mysql -uroot -pinsecure photoprism
```

#### Version Upgrade ####

If the database doesn't start properly after upgrading from an earlier MySQL or MariaDB version,
you may need to run this command in a terminal:

```bash
docker-compose exec mariadb mariadb-upgrade -uroot -p
```

Enter the MariaDB "root" password specified in your `docker-compose.yml` when prompted.

Alternatively, you can downgrade to the previous version, create a database backup using the `photoprism backup`
command, start a new database instance based on the latest version, and then restore your index with
the `photoprism restore` command.

#### Lost Root Password ####

In case you forgot the MariaDB "root" password and the one specified in your configuration does not work,
you can [start the server with the `--skip-grant-tables` flag](https://mariadb.com/docs/reference/mdb/cli/mariadbd/skip-grant-tables/)
added to the `mysqld` command in your `docker-compose.yml`. This will temporarily give full access
to all users after a restart:

```yaml
services:
  mariadb:
    command: mysqld --skip-grant-tables
```

Restart the `mariadb` service for changes to take effect:

```bash
docker-compose stop mariadb
docker-compose up -d mariadb
```

Now open a database console:

```bash
docker-compose exec mariadb mysql -uroot
```

Enter the following commands to change the password for "root":

```sql
FLUSH PRIVILEGES;
ALTER USER 'root'@'%' IDENTIFIED BY 'new_password';
ALTER USER 'root'@'localhost' IDENTIFIED BY 'new_password';
UPDATE mysql.user SET authentication_string = '' WHERE user = 'root';
UPDATE mysql.user SET plugin = '' WHERE user = 'root';
exit
```

When you are done, remove the `--skip-grant-tables` flag again to restore the original
command and restart the `mariadb` service as described above.

#### Corrupted Files ####

If your database files get corrupted frequently, it is usually because they are stored on an unreliable device such
as a USB flash drive, an SD card, or a shared network folder.

- Never use the same database files with more than one server instance
- To share a database over a network, run the database server directly on the remote server instead of sharing database files
- To repair your tables after you have moved the files to a local disk, you can [start MariaDB with `--innodb-force-recovery=1`](https://mariadb.com/kb/en/innodb-recovery-modes/), similar to how you recover a lost root password as described above