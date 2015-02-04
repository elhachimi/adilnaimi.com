---
Title: "Deploying blog using coreos docker and hugo"
Description: "Deploying blog using coreos docker and hugo."
Date: "04-02-2015"
Categories:
    - "coreos"
    - "docker"
    - "hugo"
---

#### Docker data only container

At first we need data container with templates and content for blog generation.
I used next Dockerfile:

```
FROM debian:jessie

RUN apt-get update && apt-get install --no-install-recommends -y ca-certificates git-core
RUN apt-get clean && rm -rf /var/lib/apt/lists/* /tmp/* /var/tmp/*
RUN git clone http://github.com/adilroot/adilnaimi.com.git /src
VOLUME ["/src/blog"]
WORKDIR /src
ENTRYPOINT ["git"]
CMD ["pull"]
```
There is no magic here: just clone repo to `/src` (it will be used below),
and update it on container start.

Build image:

```
docker build -t blog/content .
```

Create data container:

```
docker run --name blog_content blog/content
```

T update content and templates from github we need just:

```
docker start blog_content
```

#### Hugo

[Hugo](http://hugo.spf13.com) -- very fast static site generator, written in Go (so
many cool things written in Go btw).

Idea is to run hugo in docker container so it reads contents from one directory
and writes generated blog to another.

Hugo dockerfile:

```
FROM crosbymichael/golang

#FROM golang:1.3-onbuild

RUN apt-get update && apt-get install --no-install-recommends -y bzr

RUN go get github.com/spf13/hugo

VOLUME ["/var/www/blog"]

ENTRYPOINT ["hugo"]
CMD ["-w", "-s", "/src/blog", "-d", "/var/www/blog"]
```
So, here we `go get` hugo and use /src(remember this from content container?)
as source directory for it and `/var/www/blog` as destination.

Now build image and run container with hugo:

```
docker build -t blog/hugo .
docker run --name blog --volumes-from blog_content blog/hugo
```

Here the trick with
[`--volumes-from`](http://docs.docker.io/use/working_with_volumes/) -- we used
`/src` from `blog_content` container, and yeah, we're going to use
`/var/www/blog` from `blog` container.

#### Nginx

So, now we have container with templates and content `blog_content`, content
with ready to use blog `blog`, it's time to show this blog to the world.

I write simple config for nginx:

```
server {
    listen 80;
    server_name adilnaimi.com;
    location / {
        root /var/www/blog;
    }
}
```

Put it to sites-enabled directory, which used in this pretty dockerfile:

```
FROM dockerfile/nginx

ADD sites-enabled /etc/nginx/sites-enabled/default
```

Build image and run container with nginx:

```
docker build -t blog/nginx .
docker run -p 80:80 -d --name=nginx --volumes-from=blog blog/nginx
```

That's it, now blog is running on [adilnaimi.com](http://adilnaimi.com) and
you can read it :) I can update it just with 
```
docker start blog_content
```
