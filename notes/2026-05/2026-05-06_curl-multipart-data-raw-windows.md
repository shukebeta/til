# `--data-raw "$VAR"` silently drops multipart body on Windows Git Bash

When posting multipart form data via curl in a bash script, it's tempting to build the body in a variable and pass it with `--data-raw "$DATA"`. On Linux this often works fine. On Windows Git Bash, it silently breaks — the server receives the request but fields like `body` come through empty.

Two things go wrong at once. First, Windows-style paths (`C:\Users\...`) passed to bash utilities like `tail` and `file` are misread — the backslashes get misinterpreted and the file read returns nothing. Fix that with `cygpath`:

```bash
filename=$(cygpath -u "$1" 2>/dev/null || echo "$1")
```

Second, even with the right path, storing the body in a shell variable and passing it via `--data-raw` is unreliable. Shell variable expansion, CRLF handling, and platform-specific curl behavior all interact badly. The body gets mangled before it reaches the wire.

The fix is to bypass variables entirely: write the multipart payload to a temp file and send it with `--data-binary @file`. Since the post body is already in a file, `tail` it directly into the temp file instead of slurping it into a variable:

```bash
TMPBODY=$(mktemp)
TMPDATA=$(mktemp)

tail -n +2 "$filename" > "$TMPBODY"

{
  printf "%s\r\n" "--${BOUNDARY}"
  printf "Content-Disposition: form-data; name=\"body\"\r\n\r\n"
  cat "$TMPBODY"
  printf "\r\n%s\r\n" "--${BOUNDARY}--"
} > "$TMPDATA"

curl ... --data-binary "@$TMPDATA"

rm -f "$TMPBODY" "$TMPDATA"
```

`--data-binary @file` sends the exact bytes on disk — no shell expansion, no line ending conversion. Works the same on Linux, macOS, and Windows Git Bash.
