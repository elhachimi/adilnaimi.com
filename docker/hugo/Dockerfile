FROM golang:1.4-onbuild

#FROM crosbymichael/golang
#FROM golang:1.4-onbuild
#FROM golang:1.3-onbuild

RUN apt-get update && apt-get install --no-install-recommends -y bzr

RUN go get github.com/spf13/hugo

VOLUME ["/var/www/blog"]

ENTRYPOINT ["hugo"]
CMD ["-w", "-s", "/src/blog", "-d", "/var/www/blog"]
