FROM lsiobase/ubuntu:bionic

ARG DEBIAN_FRONTEND="noninteractive"
ENV TZ=America/Bogota

RUN \
 echo "*** Install Build Packages ***" && \
 apt-get update && \
 apt-get install -y \
	git \
	gnupg \
	jq \
	libssl-dev \
    make && \
 echo "**** install runtime *****" && \
 curl -s https://deb.nodesource.com/gpgkey/nodesource.gpg.key | apt-key add - && \
 echo 'deb https://deb.nodesource.com/node_10.x bionic main' > /etc/apt/sources.list.d/nodesource.list && \
 echo "**** install yarn repository ****" && \
 curl -sS https://dl.yarnpkg.com/debian/pubkey.gpg | apt-key add - && \
 echo "deb https://dl.yarnpkg.com/debian/ stable main" > /etc/apt/sources.list.d/yarn.list && \
 apt-get update && \
 apt-get install -y \
	fontconfig \
	fonts-noto \
	netcat-openbsd \
	nodejs \
	yarn

WORKDIR /app
COPY . .

RUN npm install

EXPOSE 3000
CMD npm start