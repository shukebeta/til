# I Automated My Blog Publishing Pipeline with a Claude Skill and Three Shell Scripts

I write a lot of short tech notes — things I learn during daily work that are worth remembering. For years the workflow was: learn something → forget to write it down → never find it again.

Then I built a pipeline: a Claude Code skill writes the note, a script publishes it to my Chyrp Lite blog, and a cron job keeps the session alive. Now I type `/record` and it's done — note saved, blog posted, zero friction.

## The Pipeline

Three pieces, each doing one thing:

1. **`/record` skill** — Claude writes a tech note from the current conversation
2. **`pub_tech_note`** — publishes the markdown file to my blog via curl
3. **`refresh_chyrp_token`** — re-logs into the blog monthly to keep the session cookie fresh

### The Skill

The skill is a `SKILL.md` file that tells Claude how to distill a conversation into a short article. Key rules: find the one non-obvious insight, pick a title that makes someone click, write like a colleague's quick tip not documentation.

After writing the file, the skill runs `~/bin/pub_tech_note <filepath>` to publish automatically.

### The Publishing Script

Chyrp Lite has no API. The admin panel is just HTML forms. But that means you can automate it with curl — fetch the CSRF hash, build a multipart form POST, and send it.

The tricky part was getting the multipart body right:

```bash
# Use printf %s for user content — backticks in titles
# will get executed as command substitution otherwise
printf "Content-Disposition: form-data; name=\"title\"\r\n\r\n"
printf "%s\r\n" "$POST_TITLE"
```

And the line endings must be CRLF. After upgrading Chyrp, the server started enforcing the multipart spec and rejected our LF payloads with 403. Took a while to figure that one out.

### The Token Refresh

Chyrp session cookies expire in about a month. Rather than manually re-authenticating, a script does it: fetch the login page, extract the CSRF hash from the hidden field, POST credentials, and capture the new `ChyrpSession` cookie from the response.

```bash
HASH=$(echo "$LOGIN_HTML" | grep -oP 'name="hash"\s+value="\K[^"]+')
# ... POST login ...
NEW_TOKEN=$(grep -oP 'ChyrpSession=\K[^;]+' /tmp/chyrp_cookies.txt | head -1)
sed -i "s/^export CHYRP_TOKEN=.*/export CHYRP_TOKEN=${NEW_TOKEN}/" ~/.bashrc.secret
```

Cron runs it on the 1st and 28th of each month:

```
0 3 1,28 * * ~/bin/refresh_chyrp_token >> ~/logs/refresh_chyrp_token.log 2>&1
```

## Credentials

Blog credentials live in `~/.bashrc.secret` — sourced by the scripts but never committed to any repo. This isn't vault-grade security (any process running as your user can read it), but it prevents the real risks: accidental git commits and screen share leaks. For a personal dev machine, that's the right trade-off.

## Lessons Learned

- **No API? No problem.** If an admin panel uses standard HTML forms, curl can automate it. The trick is capturing one real request from your browser's DevTools and reproducing it.
- **CRLF is not optional in multipart.** The spec requires it. Some servers are lenient; Chyrp started enforcing it after an upgrade and our scripts silently broke.
- **CSRF tokens change on upgrades.** The hash is generated per installation. After upgrading Chyrp, grab a fresh one from DevTools.
- **printf format strings eat backticks.** Always use `printf "%s" "$VAR"` for user content, never embed it directly in the format string.
- **Only log success.** Our first version wrote to the log before checking if curl succeeded. A failed publish left a stale entry that blocked the next attempt. Write logs only after confirmed success.

## The Result

I learn something → ask about it in a Claude session → type `/record` → done. Note saved to disk, published to blog. The friction went from "I should write this down someday" to zero.
