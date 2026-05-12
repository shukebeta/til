# VARCHAR2(4000 CHAR) Might Not Store 4000 Characters

`VARCHAR2(4000)` means 4000 **bytes**, not characters. Most people know this. What's less obvious: `VARCHAR2(4000 CHAR)` doesn't guarantee 4000 characters either.

Under the default `MAX_STRING_SIZE=STANDARD`, the hard column cap is 4000 bytes regardless of whether you declared `BYTE` or `CHAR`. In AL32UTF8, a Chinese character takes ~3 bytes, so `VARCHAR2(4000 CHAR)` on a column storing CJK text will fail once the actual byte count exceeds 4000 — around ~1333 characters in.

To actually store 4000 CJK characters in a single VARCHAR2, the instance needs `MAX_STRING_SIZE=EXTENDED` (12c+), which raises the limit to 32767 bytes. This is not the default — not even in 19c — and it's a one-way migration that requires running `utl32k.sql` in upgrade mode. Oracle keeps it off by default precisely because it changes data dictionary behavior and breaks compatibility.

Quick check for your instance:

```sql
SELECT value FROM v$parameter WHERE name = 'max_string_size';
```

`STANDARD` = 4000-byte ceiling. `EXTENDED` = 32767-byte ceiling.
