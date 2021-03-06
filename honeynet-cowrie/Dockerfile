FROM debian:buster-slim as builder
LABEL maintainer="Ittoh Kimura <kimura.itto.kd3@gmail.com>"

ENV COWRIE_GROUP=cowrie \
    COWRIE_USER=cowrie \
    COWRIE_HOME=/cowrie

RUN groupadd -r -g 1000 ${COWRIE_GROUP} && \
    useradd -r -u 1000 -d ${COWRIE_HOME} -m -g ${COWRIE_GROUP} ${COWRIE_USER}

WORKDIR ${COWRIE_HOME}

COPY cowrie cowrie/
RUN chown -R ${COWRIE_USER}:${COWRIE_GROUP} ${COWRIE_HOME}

RUN apt-get update && \
    apt-get install -y \
    authbind \
    build-essential \
    git \
    libssl-dev \
    libffi-dev \
    libpython3-dev \
    python-virtualenv \
    python3-minimal && \
    su - ${COWRIE_USER} -c "\
    cd cowrie && \
    virtualenv --python=python3 cowrie-env && \
    . cowrie-env/bin/activate && \
    pip install --upgrade pip && \
    pip install --upgrade -r requirements.txt"

FROM debian:buster-slim as runtime
LABEL maintainer="Ittoh Kimura <kimura.itto.kd3@gmail.com>"

ENV COWRIE_GROUP=cowrie \
    COWRIE_USER=cowrie \
    COWRIE_HOME=/cowrie

RUN groupadd -r -g 1000 ${COWRIE_GROUP} && \
    useradd -r -u 1000 -d ${COWRIE_HOME} -m -g ${COWRIE_GROUP} ${COWRIE_USER}

WORKDIR ${COWRIE_HOME}

RUN apt-get update && \
    apt-get install -y \
    libssl-dev \
    libffi-dev \
    procps \
    python3 \
    supervisor

COPY supervisord.conf /etc/supervisord.conf
COPY --from=builder ${COWRIE_HOME} ${COWRIE_HOME}
RUN chown -R ${COWRIE_USER}:${COWRIE_GROUP} ${COWRIE_HOME}

USER ${COWRIE_USER}
VOLUME ["/cowrie/cowrie/var", "/cowrie/cowrie/etc/"]
ENTRYPOINT ["supervisord"]
CMD ["-c", "/etc/supervisord.conf", "-e", "critical"]
EXPOSE 2222 2223
