# Fixing corrupt sqlite databses for HomeAssistant

(Some steps stolen from [here](https://support.storj.io/hc/en-us/articles/360029309111-How-to-fix-a-database-disk-image-is-malformed-))

Check a db integrity using:

```$ sqlite3 /path/to/storage/bandwidth.db "PRAGMA integrity_check;"```

0. Stop homeassistant container (if applicable). 
```
$ docker stop hass
```

1. Backup the current db.
```
$ cp hass.db hass.db.bak
```

2. Open the db in sqlite3.
```
$ sqlite3 hass.db
```

3. Dump the db to a new file.
```
# .mode insert
# .output ./dump_all.sql
# .dump
# .exit
```

4. Remove the db file that was just dumped.
```
$ rm hass.db
```

5. Create a new db from the dumped SQL. 
```
$ sqlite3 ./hass.db ".read ./dump_all.sql"
```

6. Check integrity again to make sure that solved the issue. (see above). 

7. Restart homassistant container again (if applicable). 

