FROM nginx:1.25

ARG BUILDUID
ARG BUILDGID

RUN usermod -u ${BUILDUID} www-data && \
    groupmod -g ${BUILDGID} www-data

COPY .build/docker/nginx/conf.d /etc/nginx/conf.d
COPY .build/docker/nginx/nginx.conf /etc/nginx/nginx.conf

CMD ["nginx", "-g", "daemon off;"]
