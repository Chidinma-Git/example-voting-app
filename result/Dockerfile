FROM node:18-slim@sha256:f9ab18e354e6855ae56ef2b290dd225c1e51a564f87584b9bd21dd651838830e

RUN apt-get update && \
    apt-get upgrade -y && \
    apt-get install -y --no-install-recommends curl tini && \
    rm -rf /var/lib/apt/lists/*

WORKDIR /usr/local/app

COPY package*.json ./

RUN npm ci && \
    npm cache clean --force && \
    mv /usr/local/app/node_modules /node_modules && \
    rm -rf /usr/local/lib/node_modules/npm

COPY . .

ENV PORT=80
EXPOSE 80

USER node

ENTRYPOINT ["/usr/bin/tini", "--"]
CMD ["node", "server.js"]
