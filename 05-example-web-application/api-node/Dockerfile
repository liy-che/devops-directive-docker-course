# Dockerfile for Interpreted Language
# aka does not need to be built

# Each line of command is a new layer in the union file system

# ~ clarity & security ~
# use a specific base image with node.js installed
FROM node:19.6-alpine
# FROM ubuntu

# # combine into a single command for a single layer and avoid invalidating the cache
# RUN apt update && apt install nodejs npm -y

# ~ clarity ~
WORKDIR /usr/src/app

# set up environment
ENV NODE_ENV production

# ~ speed ~
# get app code into the container (from host system or another container image)
# change in docker context (e.g. updating source code) invalidates the cache so only copy what is needed at this step
COPY package*.json ./

# ~ speed & image size ~
# using clean install to not install dev dependencies for production
# use cache mount to avoid redownloading dependencies from internet
    # need to enable buildkit and set DOCKER_BUILDKIT=1 to use this feature
RUN --mount=type=cache,target=/usr/src/app/.npm \
  npm set cache /usr/src/app/.npm && \
  npm ci --only=production

# ~ security ~
# use non-root user
USER node
# copy the rest of the app code and make them owned by user node above
COPY --chown=node:node ./src .

# ~ clarity ~
# expose the port that the app listens on
EXPOSE 3000

# commands that will be run at runtime when container is started
CMD ["node", "index.js"]