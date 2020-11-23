#
# in a qemu source checkout do
#
#   $ mkdir -p build-docker
#   $ ./scripts/archive-source.sh build-docker/qemu-src.tar.gz
#
# then, build the image
#
#   $ docker build \
#       -t qemu-system-x86_64-static:latest \
#       -f path/to/dockerfiles/qemu-system-x86_64-static/local/Dockerfile \
#       build-docker
#

FROM debian:buster as builder

LABEL maintainer="Klaus Jensen <its@irrelevant.dk>"

ARG configure_extra=""

ENV DEBIAN_FRONTEND=noninteractive
RUN apt-get -y update && \
	apt-get install -y build-essential python3-setuptools git ninja-build \
	pkg-config libglib2.0-dev libpixman-1-dev libattr1-dev libcap-ng-dev

ADD qemu-src.tar.gz /src/qemu

RUN mkdir -p /src/qemu/build; \
	cd /src/qemu/build; \
	../configure --static --target-list=x86_64-softmmu --enable-virtfs $configure_extra; \
	make -j$(nproc); \
	make install

FROM scratch
COPY --from=builder /usr/local/bin/qemu-system-x86_64 /usr/local/bin/
COPY --from=builder /usr/local/share/qemu/ /usr/local/share/qemu/
COPY --from=builder /lib/x86_64-linux-gnu/libnss_dns-2.28.so /lib/x86_64-linux-gnu/
COPY --from=builder /lib/x86_64-linux-gnu/libnss_dns.so.2 /lib/x86_64-linux-gnu/
COPY --from=builder /etc/nsswitch.conf /etc/

ENTRYPOINT ["/usr/local/bin/qemu-system-x86_64"]
