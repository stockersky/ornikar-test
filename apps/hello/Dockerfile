FROM node
WORKDIR /app
COPY package.json yarn.lock ./
RUN yarn --version
ADD src ./src
RUN yarn install
ENTRYPOINT ["yarn", "run", "start"]
