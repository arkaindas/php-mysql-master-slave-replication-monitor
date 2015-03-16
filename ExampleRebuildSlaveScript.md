Below is a simple bash script which, if properly customized will help you to rebuild a slave server from the master by means of a specialized mysqldump.

_(are you looking for the InstallationAndStandardUseCase documentation for the PHP MySQL Master/Slave Replication Monitor?  if so, this isn't it... this is just a helpful extra.)_

We use a version of this script in our production environment when our slave gets out of sync (not too often) and it works for us.

  1. It starts by modifying the hosts file and copying it out to other servers (so that all requests to db-slave actually go to the master, while this rebuild happens).
  1. Then it creates a rebuild sql file (a couple of custom commands wrapped around a mysqldump)
  1. And executes the rebuild sql file on the slave...
  1. Once done it reverts the hosts file and sends it back.

```
#!/bin/bash
DBMASTER="10.0.1.5"
DBSLAVE="10.0.1.6"
DBPASS="mySuperSecretPass"
DUMPFILE="/tmp/rebuild-db.sql"

echo "This will rebuild the slave... "
echo "It will seriously kill all queries to the slave for a while and slow down performance on the master"
echo "@link http://events.oreilly.com/pub/a/onlamp/2005/06/16/MySQLian.html"
echo ""
echo "::backing up the hosts file"
cp /etc/hosts /etc/hosts.backup
echo "::modifying the hosts file to point db-slave to the master's IP address"
echo "$DBMASTER  db-slave" > /etc/hosts
cat /etc/hosts.backup >> /etc/hosts
echo "::copy the hosts file to other servers"
scp /etc/hosts root@otherserver1:/etc/;
scp /etc/hosts root@otherserver2:/etc/;
scp /etc/hosts root@otherserver3:/etc/;
echo ""
echo "::starting the rebuild file"
echo "STOP SLAVE;" > $DUMPFILE;
echo "::dumping master, appended to the rebuild file"
mysqldump --extended-insert --all-databases --master-data -uroot -p$DBPASS -h $DBMASTER >> $DUMPFILE;
echo "::appending 'start slave' to rebuild file"
echo "START SLAVE;" >> $DUMPFILE;
echo ""
echo "::now importing rebuild file to slave [REBUILD]"
mysql -uroot -p$DBPASS -h $DBSLAVE < $DUMPFILE;
echo "::cleanup - removing dump file: $DUMPFILE"
rm $DUMPFILE
echo "::restoring the hosts file"
cp /etc/hosts.backup /etc/hosts
echo "::copy the hosts file to other servers"
scp /etc/hosts root@otherserver1:/etc/;
scp /etc/hosts root@otherserver2:/etc/;
scp /etc/hosts root@otherserver3:/etc/;
echo "--DONE--"
echo ""
exit 0
```