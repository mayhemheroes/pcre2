FROM --platform=linux/amd64 ubuntu:22.04 AS builder

RUN apt-get update && \
    DEBIAN_FRONTEND=noninteractive apt-get install -y \
        clang \
        make \
        autoconf \
        automake \
        autoconf-archive \
        libtool \
        pkg-config

COPY . /src/pcre2
WORKDIR /src/pcre2
RUN rm -rf mayhem

RUN mkdir -p m4 && \
    libtoolize -c -f && \
    rm -rf autom4te.cache Makefile.in aclocal.m4 && \
    aclocal --force -I m4 && \
    autoconf -f && \
    autoheader -f && \
    automake -a -c -f -W no-portability && \
    ./configure --enable-fuzz-support --enable-never-backslash-C \
        --with-match-limit=1000 --with-match-limit-recursion=1000 \
        CC=clang CXX=clang++ && \
    make -j$(nproc) all

RUN mkdir -p /out && \
    clang++ -fsanitize=fuzzer \
        -o /out/pcre2_fuzzer \
        .libs/libpcre2-fuzzsupport.a .libs/libpcre2-8.a

FROM --platform=linux/amd64 ubuntu:22.04
COPY --from=builder /out/pcre2_fuzzer /out/pcre2_fuzzer
