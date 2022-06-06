FROM node:latest as builder
WORKDIR /app

COPY package.json package-lock.json ./
RUN npm install

COPY . .
RUN npm run build

FROM nginx:latest as runner
COPY nginx.config /etc/nginx/nginx.conf
COPY --from=builder /app/dist /usr/share/nginx/html