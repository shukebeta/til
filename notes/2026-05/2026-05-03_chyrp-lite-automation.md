# Automating Chyrp Lite blog posts with curl

Chyrp Lite has no API — the admin panel is the only way to create posts. But since it's just standard HTML forms, you can automate it entirely with curl. Here's the full pipeline: auto-login, extract CSRF token, create posts.

## Step 1: Auto-login to get a session cookie

The login form at `/login/` uses standard URL-encoded POST with a CSRF hash. You need to fetch the login page first to extract the hash from the hidden field, then POST credentials:

```bash
# Fetch login page, extract CSRF hash
LOGIN_HTML=$(curl -s -c /tmp/chyrp_cookies.txt https://blog.example.com/login/)
HASH=$(echo "$LOGIN_HTML" | grep -oP 'name="hash"\s+value="\K[^"]+')

# POST login
curl -s -D /tmp/chyrp_headers.txt -b /tmp/chyrp_cookies.txt -c /tmp/chyrp_cookies.txt \
  -X POST https://blog.example.com/login/ \
  -H 'content-type: application/x-www-form-urlencoded' \
  --data-raw "login=${USER}&password=${PASS}&hash=${HASH}&submit="

# Extract session token
TOKEN=$(grep -oP 'ChyrpSession=\K[^;]+' /tmp/chyrp_headers.txt /tmp/chyrp_cookies.txt | head -1)
```

## Step 2: Create a post via multipart form POST

The `add_post` endpoint expects multipart/form-data with CRLF line endings. The same CSRF hash from step 1 is used here too. Watch out for shell-special characters in your title/body — always pass user content through `printf %s` to avoid backtick expansion:

```bash
BOUNDARY='----WebKitFormBoundaryYourBoundary'
CREATED_AT=$(date +"%Y-%m-%d %H:%M:%S")

DATA=$(
  printf "%s\r\n" "--${BOUNDARY}"
  printf "Content-Disposition: form-data; name=\"title\"\r\n\r\n"
  printf "%s\r\n" "$POST_TITLE"
  printf "%s\r\n" "--${BOUNDARY}"
  printf "Content-Disposition: form-data; name=\"body_field_toolbar_upload\"; filename=\"\"\r\nContent-Type: application/octet-stream\r\n\r\n\r\n"
  printf "%s\r\n" "--${BOUNDARY}"
  printf "Content-Disposition: form-data; name=\"body\"\r\n\r\n"
  printf "%s\r\n" "$POST_BODY"
  printf "%s\r\n" "--${BOUNDARY}"
  printf "Content-Disposition: form-data; name=\"status\"\r\n\r\npublic\r\n"
  printf "%s\r\n" "--${BOUNDARY}"
  printf "Content-Disposition: form-data; name=\"slug\"\r\n\r\n\r\n"
  printf "%s\r\n" "--${BOUNDARY}"
  printf "Content-Disposition: form-data; name=\"created_at\"\r\n\r\n"
  printf "%s\r\n" "$CREATED_AT"
  printf "%s\r\n" "--${BOUNDARY}"
  printf "Content-Disposition: form-data; name=\"option[comment_status]\"\r\n\r\nopen\r\n"
  printf "%s\r\n" "--${BOUNDARY}"
  printf "Content-Disposition: form-data; name=\"tags\"\r\n\r\ntech_note\r\n"
  printf "%s\r\n" "--${BOUNDARY}"
  printf "Content-Disposition: form-data; name=\"option[category_id]\"\r\n\r\n3\r\n"
  printf "%s\r\n" "--${BOUNDARY}"
  printf "Content-Disposition: form-data; name=\"feather\"\r\n\r\ntext\r\n"
  printf "%s\r\n" "--${BOUNDARY}"
  printf "Content-Disposition: form-data; name=\"hash\"\r\n\r\nac2d25ac3fadd341a2a47fb36ab94ff228e2007e\r\n"
  printf "%s\r\n" "--${BOUNDARY}--"
)

curl -s -o /dev/null -w "%{http_code}" -X POST https://blog.example.com/admin/add_post/ \
  -H "Content-Type: multipart/form-data; boundary=${BOUNDARY}" \
  -b "ChyrpSession=$TOKEN" \
  --data-raw "$DATA"
```

A successful post returns HTTP 302.

## Gotchas we hit

- **LF vs CRLF**: Multipart form spec requires CRLF. After upgrading Chyrp, the server started enforcing this and rejected our LF-only payloads with 403.
- **CSRF hash changes on upgrade**: The site-wide hash is generated per installation. After upgrading Chyrp, grab the new hash from your browser's DevTools on any admin form.
- **Backticks in titles**: If you embed `${VAR}` directly in a printf format string, backticks get executed as command substitution. Always use `printf "%s" "$VAR"` for user content.
- **Session token expiry**: ChyrpSession cookies expire (~1 month). We automated renewal with a cron job that logs in and updates the stored token every 28 days.
