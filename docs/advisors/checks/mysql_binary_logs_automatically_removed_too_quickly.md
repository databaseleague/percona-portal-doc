# mysql binary logs automatically removed too quickly

## Description
Checks that binary logs are kept at least for one day before being purged.
See https://dev.mysql.com/doc/refman/8.0/en/replication-options-binary-log.html#sysvar_binlog_expire_logs_seconds for more information


## Rule
```
For MySQL/PS 5.7 and MariaDB
Check  if expire_logs_days >= 1
For MySQL/PS 8.0 
Check if binlog_expire_logs_seconds >= 86400 
OR binlog_expire_logs_seconds = 0 and expire_logs_days > 1
```

## Resolution
Please consider increasing binlog retention period by increasing  binlog_expire_logs_seconds/expire_logs_days 
