FROM buildpack-deps:trusty

RUN sed -i 's,archive.ubuntu.com,ftp.riken.jp/Linux,g' /etc/apt/sources.list

RUN apt-get update \
    && apt-get install -y --no-install-recommends \
        cmake \
        ninja-build

#
# Google Test
#
ENV GTEST_VERSION 1.8.0

RUN curl -fSL "https://github.com/google/googletest/archive/release-$GTEST_VERSION.tar.gz" | tar zxv \
    && cd googletest-release-$GTEST_VERSION \
    && cmake -G Ninja . -DCMAKE_BUILD_TYPE=Release . \
    && ninja install \
    && cd / \
    && rm -rf googletest-release-$GTEST_VERSION

CMD ["tail", "-f", "/dev/null"]
