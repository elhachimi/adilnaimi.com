FROM debian:jessie

RUN apt-get update && apt-get install --no-install-recommends -y ca-certificates git-core
RUN apt-get clean && rm -rf /var/lib/apt/lists/* /tmp/* /var/tmp/*
RUN git clone http://github.com/adilroot/adilnaimi.com.git /src
VOLUME ["/src/blog"]
WORKDIR /src/blog
ENTRYPOINT ["git"]
CMD ["pull"]
