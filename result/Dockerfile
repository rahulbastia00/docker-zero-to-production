FROM node:20
WORKDIR /result
COPY package.json .
COPY package-lock.json .
RUN npm install

COPY . .
EXPOSE 4000
CMD ["node", "server.js"]