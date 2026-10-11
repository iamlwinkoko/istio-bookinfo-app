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



# Stage 1: Install application dependencies
FROM python:3.13-slim-bookworm AS builder

WORKDIR /build

COPY requirements.txt ./

RUN python -m pip install \
    --require-hashes \
    --no-cache-dir \
    --prefix=/install \
    -r requirements.txt


# Stage 2: Run unit tests
FROM python:3.13-slim-bookworm AS tester

WORKDIR /opt/microservices

COPY --from=builder /install /usr/local

COPY test-requirements.txt ./

RUN python -m pip install \
    --require-hashes \
    --no-cache-dir \
    -r test-requirements.txt

COPY productpage.py ./
COPY tests ./tests
COPY templates ./templates
COPY static ./static

RUN python -m unittest discover -s tests/unit


# Stage 3: Production runtime
FROM python:3.13-slim-bookworm AS runtime

WORKDIR /opt/microservices

COPY --from=builder /install /usr/local

# Remove packaging tools from the runtime image.
# Keep this separate from the builder stage.
RUN python -m pip uninstall -y setuptools wheel \
    && python -m pip uninstall -y pip

COPY productpage.py ./
COPY templates ./templates
COPY static ./static
COPY requirements.txt ./

ARG flood_factor
ENV FLOOD_FACTOR=${flood_factor:-0}

EXPOSE 9080

USER 1000

CMD ["gunicorn", "-b", "[::]:9080", "productpage:app", "-w", "8", "--keep-alive", "2", "-k", "gevent"]
