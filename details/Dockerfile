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


# Stage 1: Install dependencies
FROM ruby:3.4-slim-bookworm AS builder

WORKDIR /opt/microservices

ENV BUNDLE_PATH=/usr/local/bundle \
    BUNDLE_WITHOUT=development:test

COPY Gemfile Gemfile.lock ./

RUN bundle config set path /usr/local/bundle \
    && bundle install \
    && bundle check \
    && bundle exec ruby -r webrick -e "puts WEBrick::VERSION"


# Stage 2: Production runtime
FROM ruby:3.4-slim-bookworm AS runtime

WORKDIR /opt/microservices

ENV BUNDLE_PATH=/usr/local/bundle \
    BUNDLE_WITHOUT=development:test

COPY --from=builder /usr/local/bundle /usr/local/bundle

COPY Gemfile Gemfile.lock ./
COPY details.rb ./

ARG service_version
ENV SERVICE_VERSION=${service_version:-v1}

ARG enable_external_book_service
ENV ENABLE_EXTERNAL_BOOK_SERVICE=${enable_external_book_service:-false}

# Verify dependencies exist in the runtime image
RUN bundle check \
    && bundle exec ruby -r webrick -e "puts WEBrick::VERSION"

EXPOSE 9080

USER 1000

CMD ["bundle", "exec", "ruby", "details.rb", "9080"]
