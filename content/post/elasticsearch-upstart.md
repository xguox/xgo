---
title: "Elasticsearch Upstart"
date: 2016-05-18T16:01:23+08:00
draft: false
tags: ["Elasticsearch"]
categories: ["Elasticsearch"]
---

Elasticsearch(2.3.1) Upstart Script

```ruby
# Elasticsearch Upstart Script
description "Elasticsearch upstart script"


start on (net-device-up
          and local-filesystems
          and runlevel [2345]
          and startup)
stop on runlevel [016]

respawn
respawn limit 10 30

# NB: Upstart scripts do not respect
# /etc/security/limits.conf, so the open-file limits
# settings need to be applied here.
limit nofile 32000 32000


pre-start script
  NAME=elasticsearch
  # Elasticsearch PID file directory
  PID_DIR="/var/run/elasticsearch"
  PID_FILE="$PID_DIR/$NAME.pid"

  # Ensure that the PID_DIR exists (it is cleaned at OS startup time)
  if [ -n "$PID_DIR" ] && [ ! -e "$PID_DIR" ]; then
    mkdir -p "$PID_DIR" && chown "$NAME":"$NAME" "$PID_DIR"
  fi
  if [ -n "$PID_FILE" ] && [ ! -e "$PID_FILE" ]; then
    touch "$PID_FILE" && chown "$NAME":"$NAME" "$PID_FILE"
  fi
end script

script
  exec start-stop-daemon --start --chuid elasticsearch --exec /usr/bin/java -- -Xms256m -Xmx1g -Djava.awt.headless=true -XX:+UseParNewGC -XX:+UseConcMarkSweepGC -XX:CMSInitiatingOccupancyFraction=75 -XX:+UseCMSInitiatingOccupancyOnly -XX:+HeapDumpOnOutOfMemoryError -XX:+DisableExplicitGC -Dfile.encoding=UTF-8 -Djna.nosys=true -Des.path.home=/usr/share/elasticsearch -cp /usr/share/elasticsearch/lib/elasticsearch-2.3.1.jar:/usr/share/elasticsearch/lib/* org.elasticsearch.bootstrap.Elasticsearch start -d -p /var/run/elasticsearch/elasticsearch.pid --default.path.home=/usr/share/elasticsearch --default.path.logs=/var/log/elasticsearch --default.path.data=/var/lib/elasticsearch --default.path.conf=/etc/elasticsearch
end script

```