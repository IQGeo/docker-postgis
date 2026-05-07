#
# NOTE: THIS DOCKERFILE IS GENERATED VIA "make update"! PLEASE DO NOT EDIT IT DIRECTLY.
#

FROM docker.io/postgres:16-bullseye

ARG NPROC=

LABEL maintainer="PostGIS Project - https://postgis.net" \
      org.opencontainers.image.description="PostGIS 3.4.3 spatial database extension with PostgreSQL 16 bullseye" \
      org.opencontainers.image.source="https://github.com/postgis/docker-postgis"

ENV POSTGIS_VERSION=3.4.3
ENV POSTGIS_SHA256=802e1626252d12ec1d29261b95bf62930e1859587e44cad28d65b897a8d9ee6b

RUN set -eux \
    && apt-get update \
    # runtime deps (mark as explicitly installed so autoremove keeps them)
    && apt-get install -y --no-install-recommends \
        ca-certificates \
        libgdal28 \
        libgeos-c1v5 \
        libjson-c5 \
        libpcre2-8-0 \
        libproj19 \
        libprotobuf-c1 \
        libsfcgal1 \
        libxml2 \
    # build deps
    && apt-get install -y --no-install-recommends \
        autoconf \
        automake \
        build-essential \
        g++ \
        gettext \
        git \
        libgdal-dev \
        libgeos-dev \
        libjson-c-dev \
        libpcre2-dev \
        libproj-dev \
        libprotobuf-c-dev \
        libsfcgal-dev \
        libtool \
        libxml2-dev \
        make \
        perl \
        pkg-config \
        postgresql-server-dev-$PG_MAJOR \
        proj-bin \
        protobuf-c-compiler \
        wget \
        xsltproc \
    \
    && wget -O postgis.tar.gz "https://github.com/postgis/postgis/archive/${POSTGIS_VERSION}.tar.gz" \
    && echo "${POSTGIS_SHA256} *postgis.tar.gz" | sha256sum -c - \
    && mkdir -p /usr/src/postgis \
    && tar \
        --extract \
        --file postgis.tar.gz \
        --directory /usr/src/postgis \
        --strip-components 1 \
    && rm postgis.tar.gz \
    \
    && cd /usr/src/postgis \
    && ./autogen.sh \
    && ./configure \
        --enable-lto \
    && make -j${NPROC:-$(nproc)} \
    && make install \
    \
# This section is for refreshing the proj data for the regression tests.
# It serves as a workaround for an issue documented at https://trac.osgeo.org/postgis/ticket/5316
# This increases the Docker image size by about 1 MB.
    && projsync --system-directory --file ch_swisstopo_CHENyx06_ETRS \
    && projsync --system-directory --file us_noaa_eshpgn \
    && projsync --system-directory --file us_noaa_prvi \
    && projsync --system-directory --file us_noaa_wmhpgn \
# This section performs a regression check.
    && mkdir /tempdb \
    && chown -R postgres:postgres /tempdb \
    && su postgres -c 'pg_ctl -D /tempdb init' \
    && su postgres -c 'pg_ctl -D /tempdb -c -l /tmp/logfile -o '\'''-F'\''' start' \
    && cd regress \
    && make -j${NPROC:-$(nproc)} check RUNTESTFLAGS="--extension --verbose" PGUSER=postgres \
    \
    && su postgres -c 'psql    -c "CREATE EXTENSION IF NOT EXISTS postgis;"' \
    && su postgres -c 'psql    -c "CREATE EXTENSION IF NOT EXISTS postgis_raster;"' \
    && su postgres -c 'psql    -c "CREATE EXTENSION IF NOT EXISTS postgis_sfcgal;"' \
    && su postgres -c 'psql    -c "CREATE EXTENSION IF NOT EXISTS fuzzystrmatch; --needed for postgis_tiger_geocoder "' \
    && su postgres -c 'psql    -c "CREATE EXTENSION IF NOT EXISTS address_standardizer;"' \
    && su postgres -c 'psql    -c "CREATE EXTENSION IF NOT EXISTS address_standardizer_data_us;"' \
    && su postgres -c 'psql    -c "CREATE EXTENSION IF NOT EXISTS postgis_tiger_geocoder;"' \
    && su postgres -c 'psql    -c "CREATE EXTENSION IF NOT EXISTS postgis_topology;"' \
    && su postgres -c 'psql -t -c "SELECT version();"'              >> /_pgis_full_version.txt \
    && su postgres -c 'psql -t -c "SELECT PostGIS_Full_Version();"' >> /_pgis_full_version.txt \
    && su postgres -c 'psql -t -c "\\dx"' >> /_pgis_full_version.txt \
    \
    && su postgres -c 'pg_ctl -D /tempdb --mode=immediate stop' \
    && rm -rf /tempdb /tmp/logfile /tmp/pgis_reg \
# remove build deps
    && apt-get remove -y --purge \
        autoconf automake build-essential g++ gettext git \
        libgdal-dev libgeos-dev libjson-c-dev libpcre2-dev libproj-dev \
        libprotobuf-c-dev libsfcgal-dev libtool libxml2-dev \
        make perl pkg-config protobuf-c-compiler wget xsltproc \
    && apt-get autoremove -y \
    && rm -rf /var/lib/apt/lists/* /usr/src/postgis \
# At the end of the build, we print the collected information
# from the '/_pgis_full_version.txt' file. This is for experimental and internal purposes.
    && cat /_pgis_full_version.txt

RUN mkdir -p /docker-entrypoint-initdb.d
COPY ./initdb-postgis.sh /docker-entrypoint-initdb.d/10_postgis.sh
COPY ./update-postgis.sh /usr/local/bin
