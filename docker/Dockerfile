FROM node:alpine AS builder
COPY package.json .npmrc pnpm-lock.yaml /tmp/
RUN npm i -g pnpm && cd /tmp/ && pnpm install --prod


FROM python:3.10-alpine

ARG QL_MAINTAINER="kangwenhang"
LABEL maintainer="${QL_MAINTAINER}"
ARG QL_URL=https://github.com/${QL_MAINTAINER}/qinglong.git
ARG QL_BRANCH=develop

ENV PNPM_HOME=/root/.local/share/pnpm \
    PATH=/usr/local/sbin:/usr/local/bin:/usr/sbin:/usr/bin:/sbin:/bin:/root/.local/share/pnpm:/root/.local/share/pnpm/global/5/node_modules:$PNPM_HOME \
    LANG=zh_CN.UTF-8 \
    SHELL=/bin/bash \
    PS1="\u@\h:\w \$ " \
    QL_DIR=/ql \
    QL_BRANCH=${QL_BRANCH}

RUN set -x \
    && sed -i 's/dl-cdn.alpinelinux.org/mirrors.aliyun.com/g' /etc/apk/repositories \
    && apk update -f \
    && apk upgrade \
    && apk --no-cache add -f bash \
                             coreutils \
                             moreutils \
                             git \
                             curl \
                             wget \
                             tzdata \
                             perl \
                             openssl \
                             nginx \
                             nodejs \
                             jq \
                             openssh \
                             npm \
    && rm -rf /var/cache/apk/* \
    && apk update \
    && ln -sf /usr/share/zoneinfo/Asia/Shanghai /etc/localtime \
    && echo "Asia/Shanghai" > /etc/timezone \
    && git config --global user.email "qinglong@@users.noreply.github.com" \
    && git config --global user.name "qinglong" \
    && git config --global http.postBuffer 524288000 \
    && npm install -g pnpm \
    && mkdir data \
    && cd data \
    && mkdir config log db scripts jbot deps raw repo \
    && cd ${QL_DIR} \
    && ln -sf /${QL_DIR}/data/config /${QL_DIR} \
    && ln -sf /${QL_DIR}/data/db /${QL_DIR} \
    && ln -sf /${QL_DIR}/data/deps /${QL_DIR} \
    && ln -sf /${QL_DIR}/data/log /${QL_DIR} \
    && ln -sf /${QL_DIR}/data/scripts /${QL_DIR} \
    && ln -sf /${QL_DIR}/data/raw /${QL_DIR} \
    && ln -sf /${QL_DIR}/data/repo /${QL_DIR} \
    && ln -sf /${QL_DIR}/data/jbot /${QL_DIR} \
    && pnpm add -g pm2 ts-node typescript tslib \
    && rm -rf /root/.pnpm-store \
    && rm -rf /root/.local/share/pnpm/store \
    && rm -rf /root/.cache \
    && rm -rf /root/.npm

ARG SOURCE_COMMIT
RUN git clone -b ${QL_BRANCH} ${QL_URL} ${QL_DIR} \
    && cd ${QL_DIR} \
    && cp -f .env.example .env \
    && chmod 777 ${QL_DIR}/shell/*.sh \
    && chmod 777 ${QL_DIR}/docker/*.sh \
    && git clone -b ${QL_BRANCH} https://github.com/${QL_MAINTAINER}/qinglong-static.git /static \
    && mkdir -p ${QL_DIR}/static \
    && cp -rf /static/* ${QL_DIR}/static \
    && rm -rf /static
COPY --from=builder /tmp/node_modules/. /ql/node_modules/

WORKDIR ${QL_DIR}
    
ENTRYPOINT ["./docker/docker-entrypoint.sh"]
