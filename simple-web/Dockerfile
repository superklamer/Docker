# Base image
FROM node:alpine

WORKDIR /usr/app

# Dependencies
COPY ./package.json ./
RUN npm install
COPY ./index.js ./

# Default command
CMD ["npm", "start"]