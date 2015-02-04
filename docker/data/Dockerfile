FROM debian:jessie

RUN apt-get update && apt-get install --no-install-recommends -y ca-certificates git-core
RUN git clone http://github.com/adilroot/adilnaimi.com.git /tempo
RUN mv /tempo/blog/* /src/
VOLUME ["/src"]
WORKDIR /src
ENTRYPOINT ["git"]
CMD ["pull"]
