# Copyright Istio Authors
#
#   Licensed under the Apache License, Version 2.0 (the "License");
#   you may not use this file except in compliance with the License.
#   You may obtain a copy of the License at
#
#       http://www.apache.org/licenses/LICENSE-2.0
#
#   Unless required by applicable law or agreed to in writing, software
#   distributed under the License is distributed on an "AS IS" BASIS,
#   WITHOUT WARRANTIES OR CONDITIONS OF ANY KIND, either express or implied.
#   See the License for the specific language governing permissions and
#   limitations under the License.

# Build stage
FROM node:24-bookworm-slim AS builder

WORKDIR /opt/microservices

COPY package.json ./
RUN npm install --omit=dev

COPY ratings.js ./

# Runtime stage
FROM node:24-bookworm-slim

RUN apt-get update \
    && apt-get upgrade -y \
    && apt-get install -y --no-install-recommends curl ca-certificates \
    && update-ca-certificates \
    && rm -rf /var/lib/apt/lists/* \
    && rm -rf /usr/local/lib/node_modules/npm \
              /usr/local/lib/node_modules/corepack \
              /opt/yarn-v*

ARG service_version
ENV SERVICE_VERSION=${service_version:-v1}

WORKDIR /opt/microservices

COPY --from=builder /opt/microservices/node_modules ./node_modules
COPY --from=builder /opt/microservices/ratings.js ./ratings.js

EXPOSE 9080

USER 1000

CMD ["node", "/opt/microservices/ratings.js", "9080"]