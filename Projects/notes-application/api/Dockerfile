# stage one
FROM node:lts-alpine as builder

# install dependencies for node-gyp
RUN apk add --update --no-cache curl jq py3-configobj py3-pip py3-setuptools python3 python3-dev
RUN apk add --no-cache python3 make g++

WORKDIR /app

COPY ./package.json .
# RUN npm install --only=production
RUN npm install --production --force

# stage two
FROM node:lts-alpine

EXPOSE 3000
ENV NODE_ENV=production

USER node
RUN mkdir -p /home/node/app
WORKDIR /home/node/app

COPY . .
COPY --from=builder /app/node_modules  /home/node/app/node_modules

CMD [ "node", "bin/www" ]