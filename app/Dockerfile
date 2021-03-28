# Heavily inspired by:
# https://www.caktusgroup.com/blog/2017/03/14/production-ready-dockerfile-your-python-django-app/

FROM python:3.8-slim

# Create a group and user to run the app
ARG APP_USER=appuser
RUN groupadd -r ${APP_USER} && useradd --no-log-init -r -g ${APP_USER} ${APP_USER}

# Install packages needed to run your application (not build deps):
#   mime-support -- for mime types when serving static files
# We need to recreate the /usr/share/man/man{1..8} directories first because
# they were clobbered by a parent image.
#RUN set -ex \
#    && RUN_DEPS=" \
#    libpcre3 \
#    mime-support \
#    " \
#    && seq 1 8 | xargs -I{} mkdir -p /usr/share/man/man{} \
#    && apt-get update && apt-get install -y --no-install-recommends $RUN_DEPS \
#    && rm -rf /var/lib/apt/lists/*

ADD requirements.txt /requirements.txt

# Install build deps, then run `pip install`, then remove unneeded build deps all in a single step.
RUN set -ex \
    && BUILD_DEPS=" \
    build-essential \
    libpcre3-dev \
    " \
    && apt-get update && apt-get install -y --no-install-recommends $BUILD_DEPS \
    && pip install --no-cache-dir -r /requirements.txt \
    \
    && apt-get purge -y --auto-remove -o APT::AutoRemove::RecommendsImportant=false $BUILD_DEPS \
    && rm -rf /var/lib/apt/lists/*

# Copy the application code to the container
RUN mkdir /code/
WORKDIR /code/
ADD . /code/

# gunicorn will listen on this port
EXPOSE 8000

# Number of gunicorn workers and threads per worker
ENV WSGI_WORKERS=4 WSGI_THREADS=2

# give perms to /code/ so user can create sqlite db
RUN chown -R ${APP_USER}:${APP_USER} /code/

# Change to a non-root user
USER ${APP_USER}:${APP_USER}


ENTRYPOINT ["/code/docker-entrypoint.sh"]

# Start gunicorn
CMD gunicorn app.wsgi:application --bind 0.0.0.0:8000 \
     --workers=$WSGI_WORKERS --threads=$WSGI_THREADS \
     --statsd-host=statsd_exporter:9125
 
