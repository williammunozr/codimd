FROM node:12.19.0-alpine

# Build arguments to change source url, branch or tag
ARG CODIMD_REPOSITORY=https://github.com/williammunozr/codimd.git
ARG VERSION=master
ARG UID=10000

# Set some default config variables
ENV DOCKERIZE_VERSION=v0.6.1
ENV NODE_ENV=production

# Disable PDF export on alpine
# PhantomJS is broken on alpine and crashes CodiMD
ENV CMD_ALLOW_PDF_EXPORT=false

RUN apk add --no-cache --virtual .download \
      ca-certificates \
      wget && \
    wget https://github.com/jwilder/dockerize/releases/download/$DOCKERIZE_VERSION/dockerize-alpine-linux-amd64-$DOCKERIZE_VERSION.tar.gz && \
    tar -C /usr/local/bin -xzvf dockerize-alpine-linux-amd64-$DOCKERIZE_VERSION.tar.gz && \
    rm dockerize-alpine-linux-amd64-$DOCKERIZE_VERSION.tar.gz && \
    apk del .download

# Add configuraton files
WORKDIR /codimd
COPY . .

# Install all dependencies and build project
RUN apk add --no-cache --virtual .dep \
      bash \
      build-base \
      git \
      jq \
      openssl-dev \
      python && \
    apk add --no-cache --no-progress --repository http://dl-cdn.alpinelinux.org/alpine/edge/testing/ \
      gosu && \
    \
    # Install NPM dependencies and build project
    yarn install --pure-lockfile && \
    yarn install --production=false --pure-lockfile && \
    npm run build && \
    \
    # Clean up this layer
    yarn install && \
    yarn cache clean && \
    apk del .dep && \
    rm -rf .git && \
    \
    adduser -u $UID -h /codimd/ -D -S codimd && \
    chown -R codimd /codimd/

EXPOSE 3000

COPY ["deployments/docker-entrypoint.sh", "/usr/local/bin/docker-entrypoint.sh"]
ENTRYPOINT ["/usr/local/bin/docker-entrypoint.sh"]

CMD ["node", "app.js"]
