FROM node:16-alpine

WORKDIR /app

COPY package*.json ./
RUN npm ci --only=production && \
    apk add --no-cache netcat-openbsd

COPY server.js ./

EXPOSE 3000

CMD ["node", "server.js"]
