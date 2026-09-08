---
name: web-share
description: Publish local files, prototypes, reports, and static sites to the public share.bdgn.me site through its private S3 endpoint. Use when the user asks to publish, host, upload, or share a non-secret artifact and wants a public URL.
compatibility: Requires the AWS CLI, configured webshare credentials, and LAN or VPN access to s3.bdgn.me.
---

# Web share

Publish objects to the `web-share` bucket through `https://s3.bdgn.me`. Readers use `https://share.bdgn.me`.

## Safety rules

Everything in this bucket is world-readable. Treat an unguessable URL as obscurity, not access control.

Before uploading:

1. Inspect the exact file or directory being published.
2. Refuse to upload credentials, private notes, personal data, `.env` files, private keys, cloud configuration, or source trees that may contain them.
3. For a directory, publish the built output such as `dist/` or `public/`, not the project root.
4. Do not print, inspect, or expose the configured AWS credentials.
5. Check whether the destination prefix already contains objects. Do not overwrite an unrelated publication.

Do not use `aws s3 sync --delete` unless the user explicitly asks to replace the remote publication. The bucket has a 30 GiB quota.

## Choose the destination

Use a short URL-safe project prefix made from lowercase letters, numbers, and hyphens. Reuse a prefix when the user is updating that publication. If discovery by strangers matters, explain that the site is public and use a long random suffix only after the user accepts that limitation.

Check a proposed prefix before uploading:

```bash
aws --endpoint-url https://s3.bdgn.me s3 ls "s3://web-share/<project>/" --recursive
```

If objects exist and the user did not ask to update them, choose another prefix or ask before continuing.

## Publish one file

Keep the filename unless the user asks for a clean directory URL. Determine the MIME type and store it explicitly when the type is reliable:

```bash
file --brief --mime-type -- "<local-file>"
aws --endpoint-url https://s3.bdgn.me s3 cp \
  "<local-file>" "s3://web-share/<project>/<filename>" \
  --content-type "<mime-type>" --no-progress
```

The public URL is:

```text
https://share.bdgn.me/<project>/<filename>
```

For a clean `https://share.bdgn.me/<project>/` URL, upload an HTML page as `<project>/index.html` with content type `text/html`.

## Publish a static site

Find the actual web root. It should normally contain `index.html`. Preview the transfer first, then sync the directory contents into the project prefix:

```bash
aws --endpoint-url https://s3.bdgn.me s3 sync \
  "<build-directory>/" "s3://web-share/<project>/" \
  --dryrun

aws --endpoint-url https://s3.bdgn.me s3 sync \
  "<build-directory>/" "s3://web-share/<project>/" \
  --no-progress
```

Review the dry-run list for accidental files before running the second command. The AWS CLI infers content types from extensions. For extensionless files or a bad inference, upload those objects again with an explicit `--content-type`.

Site behavior:

- `/<project>/` and `/<project>` serve `<project>/index.html`.
- A nested directory serves its own `index.html` when present.
- A missing path under a project falls back to `<project>/index.html`, so client-side SPA routes work.
- The bucket root has no directory listing and returns 404.

## Verify and report

List the uploaded objects and test the public URL:

```bash
aws --endpoint-url https://s3.bdgn.me s3 ls \
  "s3://web-share/<project>/" --recursive
curl --fail --silent --show-error --head \
  "https://share.bdgn.me/<project>/"
```

For a single file, test its full filename URL instead. If needed, inspect stored metadata:

```bash
aws --endpoint-url https://s3.bdgn.me s3api head-object \
  --bucket web-share --key "<project>/<filename>" \
  --query ContentType --output text
```

Return the public URL and say what local path was uploaded. Mention any overwritten objects or verification failure. Pages send `X-Robots-Tag: noindex, nofollow`, but they remain public to anyone with the URL.

## List or remove publications

```bash
aws --endpoint-url https://s3.bdgn.me s3 ls s3://web-share/
aws --endpoint-url https://s3.bdgn.me s3 ls "s3://web-share/<project>/" --recursive
aws --endpoint-url https://s3.bdgn.me s3 rm "s3://web-share/<project>/<file>"
aws --endpoint-url https://s3.bdgn.me s3 rm "s3://web-share/<project>/" --recursive
```

Show what will be removed and get confirmation before recursive deletion unless the user's request already names that exact project and clearly asks to delete it.


