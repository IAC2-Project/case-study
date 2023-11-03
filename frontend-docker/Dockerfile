FROM alpine/git as clone
WORKDIR /app
RUN git clone https://github.com/IAC2-Project/iacmf-ui

FROM node:14.15.4 as node
ARG BACKEND_URL=localhost
ENV BACKEND_URL=${BACKEND_URL}
WORKDIR /app
COPY --from=clone /app/iacmf-ui /app
RUN npm install
RUN npm run build --prod

# Serve Application using Nginx Server
FROM nginx:alpine
COPY --from=node /app/dist/iacmf-ui/ /usr/share/nginx/html
EXPOSE 80