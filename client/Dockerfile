FROM node:14.21-slim
WORKDIR /app
COPY ./client/package*.json ./
ADD https://github.com/jwilder/dockerize/releases/download/v0.7.0/dockerize-alpine-linux-amd64-v0.7.0.tar.gz dockerize.tar.gz
RUN tar -xzf dockerize.tar.gz
RUN chmod +x dockerize
RUN yarn install
COPY ./client/ .
EXPOSE 3000
ENTRYPOINT ["yarn", "run", "start"]