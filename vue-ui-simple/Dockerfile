FROM nginx:1.18.0

COPY ./dist /data

RUN rm /etc/nginx/conf.d/default.conf

ADD lapis-ui.conf /etc/nginx/conf.d/

RUN /bin/bash -c 'echo init ok'
