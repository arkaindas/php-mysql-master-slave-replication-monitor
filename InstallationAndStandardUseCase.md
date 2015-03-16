# Introduction #

This assumes that you have setup and configured MySQL for Master/Slave Replication already, and have fired up the replication manually and loaded data into the slave. [instructions](http://www.howtoforge.com/mysql_database_replication)

## Use Case: Cron Job to Monitor and Repair MySQL for Master/Slave Replication ##

If you've already got a cluster/replication setup, you'll want something running all the time to ensure that it's working.  You can install this utility to manually verify that it works, and then we can automate it via a cron job. (instructions for a Linux server, Windows people should be able to figure out their variation)

### Install and Configuration ###

Download the code, and import the database/table used for testing replication.
```
svn checkout http://php-mysql-master-slave-replication-monitor.googlecode.com/svn/trunk/ /var/www/replication-monitor
chmod 755 /var/www/replication-monitor -R
mysql -uroot -p -hdb-master < /var/www/replication-monitor/create.util_replication.sql
```

NOTE: you might need to import the replication code onto your slave if it's not already replicating correctly... get that setup and working first!  Also, if you are only replicating specific databases, you will need to add util\_replication as another database to replicate.

SUGGESTION: Secure your installation with an .htaccess/.htpasswd file, or perhaps restrict the IPs which can get to the folder... Exposing something which can break your replication is a terrible idea. **this is meant to be non-public / internal-only**

### Setup the Monitor For Manual Mode ###

The package comes with an `index.php` file already, which you should probably use for manual testing... just change the configuration array in that file:
```
$replication->startup(array(
	'master' => array(
		'host' => '10.0.0.1',
		'user' => 'root',
		'pass' => 'MySuperSecretPassword',
		'testDB' => 'util_replication'
	),
	'slave' => array(
		'host' => '10.0.0.2',
		'user' => 'root',
		'pass' => 'MySuperSecretPassword',
		'testDB' => 'util_replication'
	),
));
```

### Test the Monitor ###

This is really easy, just browse to the `replication-monitor` folder and try out the links.  Check out [more information on the actions](http://code.google.com/p/php-mysql-master-slave-replication-monitor/wiki/Help), if you need clarification.  (be careful about those _destructive_ links, they are there for testing only)

Also, the more I've played with this, especially for larger databases, the less useful any of the extra links are... the monitoring (test and heal) functionality is still perfect, but for other administration like rebuilding the slave, you're probably in need of a script. (ExampleRebuildSlaveScript)

### Setup the Cron Job ###

If manual testing works well for you, I recommend you setup a cron job to run automatically.  You can link to the `index.php` file, or you can configure the `cron.php` file to run only the chosen action.

Once setup, then you'll just need to configure a cron script to run every few minutes.

```
mkdir /etc/cron.15min
vim /etc/crontab
```
```
SHELL=/bin/bash
PATH=/sbin:/bin:/usr/sbin:/usr/bin
MAILTO=root
HOME=/

# run-parts
01 * * * * root run-parts /etc/cron.hourly
02 4 * * * root run-parts /etc/cron.daily
22 4 * * 0 root run-parts /etc/cron.weekly
42 4 1 * * root run-parts /etc/cron.monthly
02 * * * * root run-parts /etc/cron.15min
17 * * * * root run-parts /etc/cron.15min
32 * * * * root run-parts /etc/cron.15min
47 * * * * root run-parts /etc/cron.15min
```
```
vim /etc/cron.15min/replication-monitor
```
```
#!/bin/bash
cd /tmp
wget --delete-after "http://localhost/replication-monitor/cron.php"
```

### Rest Peacfully ###
Do you realize how much nicer it is, when things fix themselves?  I do, and it makes me happy.

Cheers!