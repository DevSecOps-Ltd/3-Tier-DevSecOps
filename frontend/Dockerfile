# Stage 1: Build React App
FROM node:22-alpine AS build

RUN apk update && apk upgrade --no-cache

WORKDIR /app

COPY package*.json ./
RUN npm ci

COPY . .

ARG REACT_APP_API
ENV REACT_APP_API=$REACT_APP_API

RUN npm run build


# Stage 2: Serve with Nginx
FROM nginx:alpine

RUN apk update && apk upgrade --no-cache

COPY --from=build /app/build /usr/share/nginx/html

EXPOSE 80

CMD ["nginx", "-g", "daemon off;"]