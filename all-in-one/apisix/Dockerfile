#
# Licensed to the Apache Software Foundation (ASF) under one or more
# contributor license agreements.  See the NOTICE file distributed with
# this work for additional information regarding copyright ownership.
# The ASF licenses this file to You under the Apache License, Version 2.0
# (the "License"); you may not use this file except in compliance with
# the License.  You may obtain a copy of the License at
#
#     http://www.apache.org/licenses/LICENSE-2.0
#
# Unless required by applicable law or agreed to in writing, software
# distributed under the License is distributed on an "AS IS" BASIS,
# WITHOUT WARRANTIES OR CONDITIONS OF ANY KIND, either express or implied.
# See the License for the specific language governing permissions and
# limitations under the License.
#

ARG ENABLE_PROXY=false
ARG ETCD_VERSION=v3.5.4

# Build Apache APISIX,利用官方构建的镜像
FROM apache/apisix:latest


ARG ETCD_VERSION
LABEL etcd_version="${ETCD_VERSION}"

WORKDIR /tmp

USER root

RUN wget https://github.com/etcd-io/etcd/releases/download/${ETCD_VERSION}/etcd-${ETCD_VERSION}-linux-amd64.tar.gz \
    && tar -zxvf etcd-${ETCD_VERSION}-linux-amd64.tar.gz \
    && mv etcd-${ETCD_VERSION}-linux-amd64/* /usr/bin/

WORKDIR /usr/local/apisix-dashboard

ENV PATH=$PATH:/usr/local/openresty/luajit/bin:/usr/local/openresty/nginx/sbin:/usr/local/openresty/bin

# 利用官方的apisix-dashboard镜像
COPY --from=apache/apisix-dashboard:latest /usr/local/apisix-dashboard/ ./

#COPY ./config.yaml /usr/local/apisix/conf/config.yaml
COPY ./conf.yaml /usr/local/apisix-dashboard/conf/conf.yaml

# 更新修改后的jwt-auth文件
COPY ./jwt-auth.lua /usr/local/apisix/apisix/plugins/jwt-auth.lua

EXPOSE 9180 9080 9443 2379 2380 9000

COPY ./docker-entrypoint.sh /docker-entrypoint.sh

ENTRYPOINT ["/docker-entrypoint.sh"]

CMD ["docker-start"]

CMD ["sh", "-c", "(nohup etcd >/tmp/etcd.log 2>&1 &) && sleep 10 && (/usr/local/apisix-dashboard/manager-api &) && cd /usr/local/apisix && /usr/bin/apisix init && /usr/bin/apisix init_etcd && /usr/local/openresty/bin/openresty -p /usr/local/apisix -g 'daemon off;'"]

STOPSIGNAL SIGQUIT
