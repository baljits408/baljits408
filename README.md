    return datname

    def fetch(self, pg_version):
        if float(pg_version) >= 9.2 and hasattr(self, 'post_92_query'):
        if float(pg_version) >= 9.6 and hasattr(self, 'post_96_query'):
            q = self.post_96_query
        elif float(pg_version) >= 9.2 and hasattr(self, 'post_92_query'):
            q = self.post_92_query
        else:
            q = self.query
@@ -418,6 +420,33 @@ class ConnectionStateStats(QueryStats):
             ) AS tmp2
        ON tmp.mstate=tmp2.mstate ORDER BY 1
    """
    post_96_query = """
        SELECT tmp.state AS key,COALESCE(count,0) FROM
               (VALUES ('active'),
                       ('waiting'),
                       ('idle'),
                       ('idletransaction'),
                       ('unknown')
                ) AS tmp(state)
        LEFT JOIN
             (SELECT CASE WHEN wait_event IS NOT NULL THEN 'waiting'
                          WHEN state= 'idle' THEN 'idle'
                          WHEN state= 'idle in transaction'
                          THEN 'idletransaction'
                          WHEN state = 'active' THEN 'active'
                          ELSE 'unknown' END AS state,
                     count(*) AS count
               FROM pg_stat_activity
               WHERE pid != pg_backend_pid()
               GROUP BY CASE WHEN wait_event IS NOT NULL THEN 'waiting'
                          WHEN state= 'idle' THEN 'idle'
                          WHEN state= 'idle in transaction'
                          THEN 'idletransaction'
                          WHEN state = 'active' THEN 'active'
                          ELSE 'unknown' END
             ) AS tmp2
        ON tmp.state=tmp2.state ORDER BY 1
    """


class LockStats(QueryStats):
