FROM node:20-alpine 

WORKDIR /usr/src/app 

RUN apk add --no-cache python3 make g++

COPY package\*.json ./ 

RUN npm ci --only=production 

COPY . . 

USER node 

ENTRYPOINT ["node", "server.js"]