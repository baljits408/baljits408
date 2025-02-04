pg_stat_database.tup_inserted as tup_inserted,
               pg_stat_database.tup_updated as tup_updated,
               pg_stat_database.tup_deleted as tup_deleted,
               pg_stat_database.deadlocks as deadlocks,
               pg_database_size(pg_database.datname) AS size
        FROM pg_database
        JOIN pg_stat_database
