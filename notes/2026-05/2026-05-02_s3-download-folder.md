# "Download a folder" from S3? Use `sync`, not `cp`

S3 has no real directories — what looks like a folder is just a key prefix. So there's no "download folder" operation, only "download all objects with this prefix".

`aws s3 sync` is almost always what you want:

```bash
aws s3 sync s3://your-bucket/path/to/folder/ ./local-folder/
```

It's incremental — run it again and it only downloads new or changed files. `aws s3 cp --recursive` works too, but re-downloads everything every time.

Filter with `--exclude` / `--include`:

```bash
aws s3 sync s3://your-bucket/logs/ ./logs/ --exclude "*" --include "*.log"
```
