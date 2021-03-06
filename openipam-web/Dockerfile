FROM debian:stretch

MAINTAINER Eldon Koyle <ekoyle@gmail.com>

ARG git_branch=master

ENV DEBIAN_FRONTEND noninteractive

RUN    apt-get -y update \
    && apt-get install -q -y \
        uwsgi \
        uwsgi-plugin-python \
        python-ldap \
        python-psycopg2 \
        git \
        python-setuptools \
        python-pip \
        python-mysqldb \
        build-essential \
        python-dev \
        python-coverage \
        zlib1g-dev \
        python-appdirs \
    && apt-get clean \
    && rm -rf /var/lib/apt/lists/*

RUN adduser --disabled-password --gecos="openipam_user" openipam

RUN install -d -m 2775 -o openipam -g openipam /usr/local/django-openipam

RUN echo "$git_branch" | tee /git_branch

USER openipam

RUN git clone --branch $git_branch --depth=1 https://github.com/openipam/django-openipam.git /usr/local/django-openipam  # 2

USER root

# We want the Debian psycopg and python-ldap, for simplicity
RUN cat /usr/local/django-openipam/env/requirements.txt | sed '/^## EXCLUDE FROM DOCKER ##$/,$d' > /requirements_docker.txt \
    && pip install -r /requirements_docker.txt

COPY uwsgi.ini /etc/uwsgi/uwsgi.ini
RUN chmod a+r /etc/uwsgi/uwsgi.ini

RUN install -d -m 2775 -o openipam -g openipam /var/run/uwsgi

COPY start_uwsgi.sh /usr/local/bin/
RUN chmod a+rx /usr/local/bin/start_uwsgi.sh

USER openipam

# FIXME: why is this directory referenced in STATICFILES_DIRS (openipam/conf/settings.py) if it doesn't exist?
RUN [ -d /usr/local/django-openipam/components/static_components ] || mkdir -p /usr/local/django-openipam/components/static_components

RUN cd /usr/local/django-openipam && ./manage.py collectstatic --no-input

USER root

# run as root and let uwsgi drop privs
CMD /usr/local/bin/start_uwsgi.sh

