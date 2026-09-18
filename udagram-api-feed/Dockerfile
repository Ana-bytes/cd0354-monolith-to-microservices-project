FROM node:20

WORKDIR /usr/src/app

COPY package*.json ./
COPY .npmrc ./

RUN npm install

COPY . .

EXPOSE 8080

CMD ["npm", "run", "prod"]