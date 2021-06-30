FROM arm32v7/golang:alpine as gobuild

# apk update will fail on raspbian, so, use this:
# https://github.com/docker/for-linux/issues/1196#issuecomment-816600988
# For Raspberry Pi, you can upgrade libseccomp2 to a supported version from the Debian Repos eg.
# wget http://ftp.de.debian.org/debian/pool/main/libs/libseccomp/libseccomp2_2.5.1-1_armhf.deb
# dpkg -i libseccomp2_2.5.1-1_armhf.deb

RUN apk update && \
    apk add ca-certificates gcc git make musl-dev && \
    git clone https://github.com/dimkk/focalboard && \
    cd focalboard 
    #&& \
RUN make server-linux-armv7

FROM arm32v7/node:alpine as nodebuild

RUN apk update && \
    apk add ca-certificates git && \
    git clone https://github.com/mattermost/focalboard && \
    cd focalboard/webapp && \
    npm install && npm run pack

FROM arm32v7/alpine

ARG PUID=2000
ARG PGID=2000

EXPOSE 8000/tcp 9092/tcp

VOLUME /data

RUN addgroup -g ${PGID} focalboard && \
    adduser -H -D -u ${PUID} -G focalboard focalboard

WORKDIR /opt/focalboard

COPY --from=gobuild /go/focalboard/bin/linux/focalboard-server bin/
COPY --from=nodebuild /focalboard/webapp/pack pack/
COPY --from=nodebuild /focalboard/LICENSE.txt LICENSE.txt
COPY --from=nodebuild /focalboard/docker/server_config.json config.json

RUN chown -R ${PUID}:${PGID} /opt/focalboard

USER focalboard

CMD ["/opt/focalboard/bin/focalboard-server"]
