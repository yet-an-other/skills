---
name: publish-page
description: Publish a html page or SPA stateless application on share.bdgn.me, so it available from anywhere for review and use. Put one HTML file, a static site, or a client-side app online through the s3 object store. Upload over the private S3 endpoint. Readers use the public share site. 
---

# Before start
- https access to s3.bdgn.me 
- AWS* credentials are set in environmant variables
- S3 client is aws cli.


# Publish one page
- Upload the file under project name you are working.
- Never upload to the root of the bucket, always create a folder. 
- Keep the .html extension so the client and share site use the right media type.
``` wf

``` 
```aws --endpoint-url https://s3.bdgn.me s3 cp ./page.html \
  "s3://web-share/$PROJECT/page.html" \
  --content-type "text/html; charset=utf-8"
```

- Verify the exact public object and inspect its headers:

```
```
  curl -fsSIL "https://share.bdgn.me/$PROJECT/page.html"
```
```

Your public URL is https://share.bdgn.me/$PROJECT/page.html


# Publish a full site or SPA
- Build the site locally, then sync its output directory.
- Vite, Hugo, mdBook, and similar tools all work. Make sure the output contains index.html.

``` 
  aws --endpoint-url https://s3.bdgn.me s3 sync ./dist \
  "s3://web-share/$PROJECT/"

  curl -fsSIL "https://share.bdgn.me/$PROJECT/index.html"
```
```


```
Verify the exact index.html object first. You can then test the shorter project URL and any client-side routes.
The client guesses each object's Content-Type from its extension. If it guesses wrong, upload that object again with an explicit --content-type. The share site returns stored metadata as-is.

# How public URLs map to objects
Public request  Object returned
/$PROJECT/report.html -> $PROJECT/report.html
/$PROJECT/ or /$PROJECT -> $PROJECT/index.html
/$PROJECT/notes/ -> $PROJECT/notes/index.html, when it exists
/$PROJECT/client/route -> $PROJECT/index.html as the SPA fallback
There is no directory listing. The site root at https://share.bdgn.me/ returns an error by design. Use an exact object URL when verifying an upload, then test the convenience routes you plan to share.

# Inspect, update, and remove files
```
```
    # List all projects
    aws --endpoint-url https://s3.bdgn.me s3 ls s3://web-share/

    # List one project
    aws --endpoint-url https://s3.bdgn.me s3 ls \
      "s3://web-share/$PROJECT/"

    # Delete one object
    aws --endpoint-url https://s3.bdgn.me s3 rm \
      "s3://web-share/$PROJECT/old.html"

    # Delete the whole project
    aws --endpoint-url https://s3.bdgn.me s3 rm \
      "s3://web-share/$PROJECT/" --recursive

```
The bucket has a 30 GiB quota. Uploading the same key replaces the old object.


# Rules that matter
- Never upload secrets or private material. The web-share bucket allows anonymous reads.
- Use an unguessable project name for link-only sharing. This is obscurity, not access control.
- Upload through s3.bdgn.me. The public host is read-only and accepts only GET and HEAD.
- Pages carry X-Robots-Tag: noindex, nofollow, but anyone with the URL can open them.
- Clean up stale projects. All publishers share the 30 GiB quota.
