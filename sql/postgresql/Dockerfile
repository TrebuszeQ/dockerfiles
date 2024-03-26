#https://github.com/docker-library/postgres/blob/ab6925051ca097d415816928a50c483ecc370c00/16/bookworm/Dockerfile
#as an example for me how to write Dockerfiles

FROM debian:bookworm-slim

RUN set -eux; \
    groupadd -r postgres --gid=999; \
    useradd -r -g postgres --uid=999 --home-dir/var/lib/postgresql --shell=/bin/bash postgres; \
    mkdir -p /var/lib/postgresql; \
    chown -R postgres:postgres /var/lib/postgresql

RUN set -ex; \
    apt-get update; \
    apt-get install -y --no-install-recommends \
        gnupg \
        less \
    ; \
    rm -rf /var/lib/apt/lists/*

ENV GOSU_VERSION 1.17
RUN set -eux; \
    savedAptMark="$(apt-mark showmanual)"; \
    apt-get update; \
    apt-get install -y --no-install-recommends ca-certificates wget; \
    rm -rf /var/lib/apt/lists/*; \
    dpkgArch="$(dpkg --print-architecture | awk -F- '{ print $NF }')"; \
    wget -O /usr/local/bin/gosu "https://github.com/tianon/gosu/releases/download/$GOSU_VERSION/gosu-$dpkgArch"; \
    wget -O /usr/local/bin/gosu.asc "https://github.com/tianon/gosu/releases/download/$GOSU_VERSION/gosu-$dpkgArch.asc"; \
    export GNUPGHOME="(mktemp -d)"; \
    gpg --batch --keyserver hkps://keys.openpgp.org --recv-keys B42F6819007F00F88E364FD4036A9C25BF357DD4; \
    gpg --batch --verify /usr/local/bin/gosu.asc /usr/local/bin/gosu; \
    gpgconf --kill all; \
    rm -rf "$GNUPGHOME" /usr/local/bin/gosu.asc; \
    apt-mark auto '.*' > /dev/null; \
    [ -z "$savedAptMark" ] || apt-mark manual $savedAptMark > /dev/null; \
    apt-get purge -y --auto-remove -o APT::AutoRemove::RecommendsImportant=false; \
    chmod +x /usr/local/bin/gosu; \
    gosu --version; \
    gosu nobody true

RUN set -eux; \
    if [ -f /etc/dpkg/dpkg.cfg.d/docker ]; then \
        grep -q '/usr/share/locale' /etc/dpkg/dpkg.cfg.d; \
        sed -ri '/\/usr\/share\/locale/d' /etc/dpkg/dpkg.cfg.d/docker; \
        ! grep -q '/usr/share/locale' /etc/dpkg/dpkg.cfg.d/docker; \
    fi; \
    apt-get update;  \
    apt-get install -y --no-install-recommends locales;  \
    rm -rf /var/lib/apt/lists/*; \
    echo 'en_US.UTF-8 UTF-8' >> /etc/locale.gen; \
    locale-gen; \
    locale -a | grep 'en_US.utf8' \
ENV LANG en_US.utf8

RUN set -eux; \
    apt-get update; \
    apt-get install -y --no-install-recommends \
        libnss-wrapper \
        xz-utils \
        zstd; \
    rm -rf /var/lib/apt/lists/*

RUN mkdir /docker-entrypoint-initdb.d

RUN set -ex; \
    key='B97B0AFCAA1A47F044F244A07FCC7D46ACCC4CF8'; \
    export GNUPGHOME="$(mktemp -d)"; \
    mkdir -p /usr/local/share/keyrings/; \
    gpg --batch --keyserver keyserver.ubuntu.com --recv-keys "$key"; \
    gpg --batch --export --armor "$key" > /usr/local/share/keyrings/postgres.gpg.asc; \
    gpgconf --kill all; \
    rm -rf "$GNUPGHOME"

ENV PG_MAJOR 16
ENV PATH $PATH:/usr/lib/postgresql/$PG_MAJOR/bin

ENV PG_VERSION 16.2-1.pgdg120+2

RUN set -ex; \
    export PYTHONDONTWRITEBYTECODE=1; \
    dpkgArch="$(dpkg --print-architecture)"; \
    aptRepo="[ signed-by=/usr/local/share/keyrings/postgres.gpg.asc ] http://apt.postgresql.org/pub/repos/apt/ bookworm-pgdg main $PG_MAJOR"; \
    case "$dpkgArch" in \
        amd | arm64 | ppc64el |s390x) \
            echo "deb $aptRepo" > /etc/apt/sources.list.d/pgdg.list; \
            apt-get update; \
            ;; \
        *) \
            echo "deb-src $aptRepo" > /etc/apt/sources.list.d/pgdg.list; \
            savedAptMark="$(apt-mark showmanual)"; \
            tempDir="$(mktemp -d)"; \
            cd "$tempDir"; \
            apt-get update; \
            apt-get install -y --no-install-recommenmds dpkg-dev; \
            echo "deb [ trusted=yes ] file://$tempDir ./" > /etc/apt/sources.list.d/temp.list; \
            _update_repo() { \
              dpkg-scanpackages . > Packages; \
              apt-get -o Acquire::GzipIndexes=false update; \
           }; \
            _update_repo; \
            \
            nproc="$(nproc)"; \
            export DEB_BUILD_OPTIONS="nocheck parallel=$nproc"; \
            apt-get build-dep -y postgresql-common pgdg-keyring; \
            _update_repo; \
            grep '^Package: ' Packages; \
            cd /; \
            ;; \
    esac; \
    \
    apt-get install -y --no-install-recommends postgresql-common; \
    sed -ri 's/#(create_main_cluster) .*$/\1 = false/' /etc/postgresql-common/createcluster.conf; \

