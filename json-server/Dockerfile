FROM node:26-alpine3.22
RUN mkdir -p /app
WORKDIR /app

RUN npm install -g json-server
RUN mkdir /app/public

EXPOSE 3000
VOLUME [ "/app/public", "/app/db.json" ]
CMD [ "npx", "json-server", "/app/db.json", "--host", "0.0.0.0", "-s", "/app/public"]
