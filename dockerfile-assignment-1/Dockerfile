FROM nginx:latest

WORKDIR /usr/src/app

COPY package.json package.json

RUN apk add --no-cache tini \
    && npm install
	&& npm cache clean --force
    && ln -sf /dev/stdout /var/log/nginx/access.log \
    && ln -sf /dev/stderr /var/log/nginx/error.log \

EXPOSE 3000

CMD [ "/sbin/tini", "--node", "./bin/www" ]
# NOTE: The instructions are now in the README.md file in this directory.
