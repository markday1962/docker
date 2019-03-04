FROM ubuntu:18.04
MAINTAINER Mark Day, mark@mark-day.me
ENV REFRESHED_AT 2019_03_04
ARG DEBIAN_FRONTEND=noninteractive

RUN \
    apt-get update -qq -y && \
    apt-get install -y --no-install-recommends apt-utils \ 
    nmap

CMD ["/bin/bash"]
