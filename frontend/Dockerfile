# Stage 1: Build React App
FROM node:22-alpine AS build

RUN apk update && apk upgrade

WORKDIR /app

COPY package*.json ./
RUN npm install

COPY . .

# Optional: pass REACT_APP_API as build arg
ARG REACT_APP_API
ENV REACT_APP_API=$REACT_APP_API

RUN npm run build


# Stage 2: Serve with NGINX
FROM nginx:alpine

# Copy custom NGINX configuration
COPY default.conf /etc/nginx/conf.d/default.conf

# Copy React build output from build stage
COPY --from=build /app/build /usr/share/nginx/html

# Set permissions
RUN chmod -R 755 /usr/share/nginx/html

EXPOSE 80

CMD ["nginx", "-g", "daemon off;"]