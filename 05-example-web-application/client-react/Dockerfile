# hybrid between Dockerfiles for interpreted and compiled languages
# build static files then serve using a separate deployable stage

# syntax=docker/dockerfile:1.5
# escape=\
# ^ OPTIONAL directives, must be at top if used
# syntax needed for COPY --link

FROM node:19.4-bullseye AS build

WORKDIR /usr/src/app

COPY package*.json ./

# still in build stage so no --only=production
RUN --mount=type=cache,target=/usr/src/app/.npm \
  npm set cache /usr/src/app/.npm && \
  npm ci

COPY . .

# generate static files (html, javascript, css) to dist/ folder
RUN npm run build

# running unprivileged
FROM nginxinc/nginx-unprivileged:1.23-alpine-perl

# copy in nginx config file
# use --link to avoid breaking cache if second stage base image is changed
COPY --link nginx.conf /etc/nginx/conf.d/default.conf

COPY --link --from=build usr/src/app/dist/ /usr/share/nginx/html

# running unprivileged prohibits from using port 80
EXPOSE 8080