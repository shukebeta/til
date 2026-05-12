# Suppress SQLcl Banner and Version Noise with -S

When scripting with Oracle SQLcl, the startup banner (version, copyright, connection info) clamps your output. The `-S` (silent) flag suppresses all of it:

```bash
sql -S user/password@connect_string @script.sql
```

This gives you clean output suitable for piping or log capture.

For even more control inside the session, pair it with:

```sql
set heading off
set feedback off
set pagesize 0
set echo off
```

`-S` is the entry-level switch. The `set` commands handle the rest.
