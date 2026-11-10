# Migration Testing

This is a small web page deployed as a docker container to test publishing docker images to an individual user account after moving the repository to an organization repository.

## Fieldnotes

A small static landing page served by BusyBox `httpd`.

## Run

```sh
docker build -t fieldnotes .
docker run --rm -p 8080:8080 fieldnotes
```

Open <http://localhost:8080>. The container serves plain HTTP; place it behind your existing TLS and compression layer as needed.

To edit the page, update `site/index.html` and the SVGs in `site/images/`, then rebuild the image.
