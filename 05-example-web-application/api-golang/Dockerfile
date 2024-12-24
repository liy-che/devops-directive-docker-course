# Dockerfile for Compiled Language
# aka needs to be built before running it

# using a base image with more build capabilities
# its bigger size does not matter much in multi-stage build
# in which build and deploy stages are separated
# the deployable image will be much smaller with the
# scratch (empty) base image
FROM golang:1.19-buster AS build

WORKDIR /app

# use non-root user, but probably overkill in this dockerfile
RUN useradd -u 1001 nonroot

# copy only the dependency files
COPY go.mod go.sum ./

# use cache mount to avoid redownloading dependencies from internet
RUN --mount=type=cache,target=/go/pkg/mod \
  --mount=type=cache,target=/root/.cache/go-build \
  go mod download

# use .dockerignore file or folder to avoid moving unnecessary files
COPY . .

# add flags to statically link binary
RUN go build \
  -ldflags="-linkmode external -extldflags -static" \
  -tags netgo \
  -o api-golang

# official empty image
FROM scratch

# set environment for production
ENV GIN_MODE release

# copying from the build instead of the host 
# required in order to use non-root user
COPY --from=build /etc/passwd /etc/passwd

# copy the built binary from this build to the empty image
# which will be deployed
COPY --from=build /app/api-golang api-golang

# use non-root user
user nonroot

# app listens on port 8080
EXPOSE 8080

# already built, so no need for `go run -o api-golang`
# which builts and runs it
CMD ["./api-golang"]