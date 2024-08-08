---
title: Postgres - Logging queries
---

We had an issue with some JQL queries returning weird results from the db, so we wanted to see exactly what's arriving to the psql service. To see that:

1. Edit the config file: `/var/lib/postgresql/data/postgresql.conf`

2. Unmark and change the following:
```properties
#logging_collector = off                # Enable capturing of stderr and csvlog
                                        # into log files. Required to be on for
                                        # csvlogs.
                                        # (change requires restart)

# These are only used if logging_collector is on:
#log_directory = 'pg_log'               # directory where log files are written,
                                        # can be absolute or relative to PGDATA
#log_filename = 'postgresql-%Y-%m-%d_%H%M%S.log'        # log file name pattern,
                                        # can include strftime() escapes
[...]
log_statement = 'none'                   # none, ddl, mod, all
```

* The `logging_collector` should be set to `on` to enable logging
* The  `log_statement` should be set to `all` to enable query logging
* The `log_directory` and `log_filename` can stay the same, depends on what you want.

So your line should look like:
```properties
#logging_collector = on                # Enable capturing of stderr and csvlog
                                        # into log files. Required to be on for
                                        # csvlogs.
                                        # (change requires restart)

# These are only used if logging_collector is on:
#log_directory = 'pg_log'               # directory where log files are written,
                                        # can be absolute or relative to PGDATA
#log_filename = 'postgresql-%Y-%m-%d_%H%M%S.log'        # log file name pattern,
                                        # can include strftime() escapes
[...]
log_statement = 'all'                   # none, ddl, mod, all
```

Now restart your service, and you're good to go : the logs will be at `/var/lib/postgresql/data/pg_log`

**Don't run this on production, as it will seriously fuck up your performance!**
