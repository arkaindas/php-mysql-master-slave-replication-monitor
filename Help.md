# Actions: Minimal Response #
## Test ##
This preforms a test (status and row replication testing) and returns the status: "Success" or "Failure"

`..../replication-monitor/?action=Test`

## Test & Heal ##
This preforms a test (status and row replication testing) and returns "Success" if successful.  If the test fails, it then attempts to automatically "Heal" by restarting the replication (see **Restart Slave**).  It will then test again and return the status: "Success" or "Failure"

`..../replication-monitor/?action=TestAndHeal`

# Actions: HTML Response #
## Test ##
Like Above, but with more output, links, errors, and debugging information.

`..../replication-monitor/?action=TestHTML`

## Test & Heal ##
Like Above, but with more output, links, errors, and debugging information.

`..../replication-monitor/?action=TestAndHealHTML`

## Restart Slave ##
This action will restart the Slave server
```
STOP SLAVE;
RESET SLAVE;
CHANGE MASTER TO MASTER_LOG_FILE='$file', MASTER_LOG_POS=$pos;
START SLAVE;
```
And then it will test status again, by preforming a "Skip Start Slave" action.

`..../replication-monitor/?action=RestartSlave`

## Skip Start Slave ##
Sometimes there's a conflict when replicating content from the Master to the Slave (perhaps someone wrote a record to the slave?).  This action will attempt to skip the record and start replication again... it loops 10 times, attempting.
```
SET GLOBAL SQL_SLAVE_SKIP_COUNTER=1;
START SLAVE;
START SLAVE IO_THREAD;
START SLAVE SQL_THREAD;
```
This results in a status message, testing after the start.

`..../replication-monitor/?action=SkipStartSlaveIfBadStatus`

# Actions: Testing #
## Break Replication (destructive) ##
This writes a row to the slave, which will break your replication.  Useful for testing, but be really careful in a production environment.  (you can edit the code to remove this link if you like.)

`..../replication-monitor/?action=BreakReplication`

## Rebuild Slave (destructive) ##
This will stop replication, delete all databases on the slave, and then load the data from the master.

**WARNING: this can be _really slow_ and lock up the server while running.**

When I do this, I typically will change hosts/dns to repoint "slave" to the master server so nothing is hitting the slave... then rebuild the slave... once complete, swap the hosts/dns back.  (of course this assumes you have configured the replication-monitor to access the slave server by a different means/hostname)

This runs the following on the slave:
```
STOP SLAVE;
SHOW DATABASES;
DROP DATABASE `$eachDatabase`; #excluding 'mysql' and 'information_schema'
LOAD DATA FROM MASTER;
```

`..../replication-monitor/?action=RebuildSlave`

also, you can run each sub-function of the process:
`..../replication-monitor/?action=DeleteSlave`
`..../replication-monitor/?action=LoadDataFromMaster`
`..../replication-monitor/?action=RestartSlave`