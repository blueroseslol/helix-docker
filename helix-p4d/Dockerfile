# --------------------------------------------------------------------------------
# Docker configuration for P4D
# --------------------------------------------------------------------------------

FROM ubuntu:focal

LABEL vendor="Sourcegraph"
LABEL maintainer="Joe Chen (joe@sourcegraph.com)"

# Update Ubuntu and add Perforce Package Source
RUN \
  apt-get update && \
  apt-get install -y wget gnupg2 && \
  wget -qO - https://package.perforce.com/perforce.pubkey | gpg --dearmor | tee /usr/share/keyrings/perforce.gpg && \
  echo "deb [signed-by=/usr/share/keyrings/perforce.gpg] http://package.perforce.com/apt/ubuntu focal release" > /etc/apt/sources.list.d/perforce.list && \
  apt-get update

# --------------------------------------------------------------------------------
# Docker BUILD
# --------------------------------------------------------------------------------

# Create perforce user and install Perforce Server
RUN \
  apt-get install -y helix-p4d && \
  apt-get install -y helix-swarm-triggers

# Add external files
COPY ./files/restore.sh /usr/bin/restore.sh
COPY ./files/setup.sh /usr/bin/setup.sh
COPY ./files/init.sh /usr/bin/init.sh
COPY ./files/latest_checkpoint.sh /usr/bin/latest_checkpoint.sh

RUN \
  chmod +x /usr/bin/restore.sh && \
  chmod +x /usr/bin/setup.sh && \
  chmod +x /usr/bin/init.sh && \
  chmod +x /usr/bin/latest_checkpoint.sh

# --------------------------------------------------------------------------------
# Docker ENVIRONMENT
# --------------------------------------------------------------------------------

# Default Environment
ARG NAME=perforce-server
ARG P4NAME=master
ARG P4TCP=1666
ARG P4USER=admin
ARG P4PASSWD=pass12349ers
ARG P4CASE=-C0
ARG P4CHARSET=utf8

# Dynamic Environment
ENV NAME=$NAME \
  P4NAME=$P4NAME \
  P4TCP=$P4TCP \
  P4PORT=$P4TCP \
  P4USER=$P4USER \
  P4PASSWD=$P4PASSWD \
  P4CASE=$P4CASE \
  P4CHARSET=$P4CHARSET \
  JNL_PREFIX=$P4NAME

# Base Environment
ENV P4HOME=/p4

# Derived Environment
ENV P4ROOT=$P4HOME/root \
  P4DEPOTS=$P4HOME/depots \
  P4CKP=$P4HOME/checkpoints

# Expose Perforce; TCP port and volumes
EXPOSE $P4TCP
VOLUME $P4HOME

# --------------------------------------------------------------------------------
# Docker RUN
# --------------------------------------------------------------------------------

ENTRYPOINT \
  init.sh && \
  /usr/bin/tail -F $P4ROOT/logs/log

HEALTHCHECK \
  --interval=2m \
  --timeout=10s \
  CMD p4 info -s > /dev/null || exit 1
