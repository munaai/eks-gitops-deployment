# Stage 1: Frontend (React/Vite)
FROM node:20-alpine AS frontend
WORKDIR /web
COPY web/package.json web/yarn.lock ./
RUN yarn install --frozen-lockfile
COPY web/ .
RUN yarn build

# Stage 2: Backend build (Go)
FROM golang:1.25-alpine AS backend
WORKDIR /app
RUN apk add --no-cache git
COPY . .
COPY --from=frontend /web/dist ./server/router/frontend/dist
ENV CGO_ENABLED=0
RUN go build -ldflags="-s -w" -o memos ./bin/memos

# Stage 3: Final runtime image
FROM alpine:3.19
WORKDIR /app
RUN addgroup -S memos && adduser -S memos -G memos
RUN mkdir -p /var/opt/memos && chown -R memos:memos /var/opt/memos
COPY --from=backend /app/memos /app/memos
RUN chown memos:memos /app/memos
ENV MEMOS_DATA=/var/opt/memos
USER memos
EXPOSE 8081
CMD ["/app/memos"]