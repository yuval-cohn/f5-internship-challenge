# Proxy image inherited from the previous team.

FROM ubuntu:24.04

COPY nginx.conf /etc/nginx/nginx.conf

RUN apt-get update && \
	DEBIAN_FRONTEND=noninteractive TZ="Asia/Jerusalem" apt-get install --option "Dpkg::Options::=--force-confdef" \
	--option "Dpkg::Options::=--force-confold" curl htop nginx git vim -y


EXPOSE 80

CMD ["nginx", "-g", "daemon off;"]
