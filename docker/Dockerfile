ARG BASE_IMAGE_VERSION
FROM python:$BASE_IMAGE_VERSION-slim-buster as build

RUN apt update -qq \
    && apt install git curl gcc g++ make file musl-dev libffi6 libffi-dev zlib1g zlib1g-dev -y \
    && curl -sSL https://raw.githubusercontent.com/python-poetry/poetry/master/get-poetry.py | python

WORKDIR /usr/src/app/
ADD poetry.lock pyproject.toml ./
RUN $HOME/.poetry/bin/poetry install

ADD src src
ADD config config
ADD run.py VERSION ./
RUN $HOME/.poetry/bin/poetry run pyinstaller run.py -n bbb-py-rest --clean --onefile \
    --add-data pyproject.toml:. \
    --add-data VERSION:. \
    --add-data config:config

FROM python:$BASE_IMAGE_VERSION-slim

ARG MAINTAINER="NubeIO <dev@nube-io.com>"
ARG APP_VERSION="1.0.0"
ARG RUBIX_UID=642
ARG NUBEIO_GID=1173
ARG COMMIT_SHA=$COMMIT_SHA

LABEL maintainer=$MAINTAINER version=$APP_VERSION commit=$COMMIT_SHA

ENV RUBIX_VOLUME=/data/bbb-py-rest

RUN groupadd -g $NUBEIO_GID nubeio \
    && useradd -u $RUBIX_UID -G nubeio rubix \
    && mkdir -p $RUBIX_VOLUME \
    && chown -R rubix:nubeio $RUBIX_VOLUME

WORKDIR /usr/src/app/
COPY --chown=rubix:nubeio --from=build /usr/src/app/dist/bbb-py-rest ./

USER rubix:nubeio

VOLUME $RUBIX_VOLUME

EXPOSE 5000

ENTRYPOINT [ "./bbb-py-rest" ]
