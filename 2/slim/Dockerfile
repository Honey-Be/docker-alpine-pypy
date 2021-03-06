FROM alpine:3.4
LABEL maintainer "Jamie Hewland <jhewland@gmail.com>"

# ensure local pypy is preferred over distribution python
ENV PATH /usr/local/bin:$PATH

# http://bugs.python.org/issue19846
# > At the moment, setting "LANG=C" on a Linux system *fundamentally breaks Python 3*, and that's not OK.
ENV LANG C.UTF-8

# install ca-certificates so that HTTPS works consistently
# the other runtime dependencies for PyPy are installed later
RUN apk add --no-cache ca-certificates

ENV PYPY_VERSION 5.7.0
ENV PYPY_SHA256 3b0df4c5a28260abe1986bdb78014cf0f4dface7901efbee86b1513cce6e8868

# if this is called "PIP_VERSION", pip explodes with "ValueError: invalid truth value '<VERSION>'"
ENV PYTHON_PIP_VERSION 9.0.1

RUN set -ex \
    && apk add --no-cache --virtual .fetch-deps \
        openssl \
        tar \
        xz \
    \
    && wget -O pypy.tar.bz2 "https://github.com/JayH5/alpine-pypy/releases/download/2-$PYPY_VERSION/pypy2-v${PYPY_VERSION}-linux64.tar.bz2" \
    && echo "$PYPY_SHA256  pypy.tar.bz2" | sha256sum -c - \
    && tar -xjC /usr/local --strip-components=1 -f pypy.tar.bz2 \
    && rm pypy.tar.bz2 \
    \
    && runDeps="$( \
        scanelf --needed --nobanner --recursive /usr/local \
            | awk '{ gsub(/,/, "\nso:", $2); print "so:" $2 }' \
            | sort -u \
            | grep -v 'libpypy-c' \
    )" \
    && apk add --no-cache --virtual .pypy-rundeps $runDeps \
    \
        && wget -O /tmp/get-pip.py 'https://bootstrap.pypa.io/get-pip.py' \
        && pypy /tmp/get-pip.py "pip==$PYTHON_PIP_VERSION" \
        && rm /tmp/get-pip.py \
# we use "--force-reinstall" for the case where the version of pip we're trying to install is the same as the version bundled with Python
# ("Requirement already up-to-date: pip==8.1.2 in /usr/local/lib/python3.6/site-packages")
# https://github.com/docker-library/python/pull/143#issuecomment-241032683
    && pip install --no-cache-dir --upgrade "pip==$PYTHON_PIP_VERSION" \
# then we use "pip list" to ensure we don't have more than one pip version installed
# https://github.com/docker-library/python/pull/100
    && [ "$(pip list |tac|tac| awk -F '[ ()]+' '$1 == "pip" { print $2; exit }')" = "$PYTHON_PIP_VERSION" ] \
    \
    && apk del .fetch-deps \
    && rm -rf ~/.cache

CMD ["pypy"]
