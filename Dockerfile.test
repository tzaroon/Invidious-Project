# get default docker image
#FROM crystallang/crystal:1.1.1-alpine AS builder
FROM crystallang/crystal:latest-alpine AS builder

# set build arguments
ARG release
# add some initial packages
RUN apk add --no-cache sqlite-static yaml-static

# set working directory
WORKDIR /invidious

# copy and configure shard 
COPY ./shard.yml ./shard.yml
COPY ./shard.lock ./shard.lock
RUN shards install

# copy remote libs
COPY --from=quay.io/invidious/lsquic-compiled /root/liblsquic.a ./lib/lsquic/src/lsquic/ext/liblsquic.a

# copy source file
COPY ./src/ ./src/

# TODO: .git folder is required for building – this is destructive.
# See definition of CURRENT_BRANCH, CURRENT_COMMIT and CURRENT_VERSION.
COPY ./.git/ ./.git/

RUN crystal spec --warnings all \
    --link-flags "-lxml2 -llzma"

RUN if [ ${release} == 1 ] ; then \
        crystal build ./src/invidious.cr \
        --release \
        --static --warnings all \
        --link-flags "-lxml2 -llzma"; \
    else \
        crystal build ./src/invidious.cr \
        --static --warnings all \
        --link-flags "-lxml2 -llzma"; \
    fi

# another run stage
FROM alpine:latest
ARG dbuser
ARG dbpasswd
ARG dbhost
ARG dbname
# Installing Bash
# RUN  apk add --no-cache bash
RUN sed -i 's/bin\/ash/bin\/bash/g' /etc/passwd


# set working directory
WORKDIR /invidious

# adding user and group
# RUN addgroup -g 1000 -S invidious && \
#    adduser -u 1000 -S invidious -G invidious

# copying assests folders
COPY ./assets/ ./assets/
COPY ./locales/ ./locales/
# setting up permissions to new users
COPY  ./config/config.* ./config/

# copying application config file
RUN mv config/config.example.yml config/config.yml && \
    sed -i "s/replaceable_db_user/${dbuser}/g" config/config.yml && \
    sed -i "s/replaceable_db_passwd/${dbpasswd}/g" config/config.yml && \
    sed -i "s/replaceable_db_host/${dbhost}/g" config/config.yml && \
    sed -i "s/replaceable_db_name/${dbname}/g" config/config.yml

#RUN sed -i 's/host: \(127.0.0.1\|localhost\)/host: postgres/' confi
#RUN sed -i 's/host: \(127.0.0.1\|localhost\)/host: postgres/' config/config.yml
#COPY ./config/sql/ ./config/sql/
COPY --from=builder /invidious/invidious .
RUN chmod o+rX -R ./assets ./config ./locales

# Installing and Configuring OpenSSH Server
# RUN apk add --no-cache openssh
# RUN echo "root:Docker!" | chpasswd
# COPY sshd_config /etc/ssh/

# Configure Nginx
# RUN apk add nginx && echo "daemon off;" >> /etc/nginx/nginx.conf && rm /etc/nginx/http.d/default.conf

#RUN echo "daemon off;" >> /etc/nginx/nginx.conf
# COPY default.conf /etc/nginx/http.d/
# RUN ln -sf /dev/stdout /var/log/nginx/access.log
# RUN ln -sf /dev/stderr /var/log/nginx/error.log
#RUN rm /etc/nginx/sites-enabled/Default
#RUN rm /etc/nginx/sites-available/Default
RUN echo "exit 0" > /usr/sbin/policy-rc.d

# chown the root directory:
#RUN chown -R www-data:www-data /var/www/html && \
#    find /var/www/html -type f -exec chmod 644 {} \; && \
#    find /var/www/html -type d -exec chmod 755 {} \;

# Copying Initialization Script
# COPY init.sh /invidious/init.sh
# RUN chmod 755 /invidious/init.sh

# expose required ports
EXPOSE 3000

#run entry script
#ENTRYPOINT ["/invidious/init.sh"]

# USER invidious
CMD /invidious/invidious
# CMD nginx -g "daemon off;" &&  /invidious/invidious && /etc/init.d/ssh start
