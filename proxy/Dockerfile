FROM alpine:latest

COPY nginx.conf /etc/nginx/nginx.conf

# 1. Update package index and install nginx
RUN apk update && \
    apk add --no-cache nginx

EXPOSE 8081

USER nginx
CMD ["nginx", "-g", "daemon off;"]
