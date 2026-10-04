FROM node:22-alpine

RUN apk update && apk upgrade --no-cache

WORKDIR /app

COPY package*.json ./

RUN npm ci --omit=dev \
    && rm -rf /usr/local/lib/node_modules/npm \
              /root/.npm

COPY . .

EXPOSE 5000

CMD ["node", "app.js"]