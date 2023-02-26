FROM ubuntu:20.04

# Install common dependencies
ENV DEBIAN_FRONTEND=noninteractive
RUN apt-get -y update && \
    apt-get -y install sudo \ 
    apt-utils \
    build-essential \
    openssl \
    clang \
    graphviz-dev \
    git \
    autoconf \
    libgnutls28-dev \
    libssl-dev \
    llvm \
    python3-pip \
    nano \
    net-tools \
    vim \
    gdb \
    netcat \
    strace \
    wget \
    libcap-dev

# Add a new user ubuntu, pass: ubuntu
RUN groupadd ubuntu && \
    useradd -rm -d /home/ubuntu -s /bin/bash -g ubuntu -G sudo -u 1000 ubuntu -p "$(openssl passwd -1 ubuntu)"

RUN chmod 777 /tmp

RUN pip3 install gcovr==5.2

# Use ubuntu as default username
USER ubuntu
WORKDIR /home/ubuntu

# Import environment variable to pass as parameter to make (e.g., to make parallel builds with -j)
ARG MAKE_OPT

# Set up fuzzers
RUN git clone https://github.com/pasok0n/aflnet.git && \
    cd aflnet && \
    make clean all $MAKE_OPT && \
    cd llvm_mode && make $MAKE_OPT

RUN git clone https://github.com/profuzzbench/aflnwe.git && \
    cd aflnwe && \
    make clean all $MAKE_OPT && \
    cd llvm_mode && make $MAKE_OPT

# Set up environment variables for AFLNet
ENV WORKDIR="/home/ubuntu/experiments"
ENV AFLNET="/home/ubuntu/aflnet"
ENV PATH="${PATH}:${AFLNET}:/home/ubuntu/.local/bin:${WORKDIR}"
ENV AFL_PATH="${AFLNET}"
ENV AFL_I_DONT_CARE_ABOUT_MISSING_CRASHES=1 \
    AFL_SKIP_CPUFREQ=1 \
    AFL_NO_AFFINITY=1


RUN mkdir $WORKDIR

USER root
RUN apt-get -y install dnsutils libssl-dev libuv1-dev perl pkg-config
# Set up environment variables for ASAN
env ASAN_OPTIONS='abort_on_error=1:symbolize=0:detect_leaks=0:detect_stack_use_after_return=1:detect_container_overflow=0:poison_array_cookie=0:malloc_fill_byte=0:max_malloc_fill_size=16777216'

# Download and compile Bind9 for fuzzing
RUN cd $WORKDIR && \
    wget https://downloads.isc.org/isc/bind9/9.18.9/bind-9.18.9.tar.xz && \
    tar -xf bind-9.18.9.tar.xz && mv bind-9.18.9 bind && \
    cd bind && \
    ./configure \
    CC=afl-clang-fast \
    --disable-doh \
    --prefix=$WORKDIR/bind/build && \
    AFL_USE_ASAN=1 make $MAKE_OPT && make install

# Download and compile Bind9 for coverage analysis
RUN cd $WORKDIR && \
    tar -xf bind-9.18.9.tar.xz && mv bind-9.18.9 bind-gcov && \
    cd bind-gcov && \
    ./configure \
    CC="gcc" \
    CFLAGS="-g -O0 -fprofile-arcs -ftest-coverage" \
    LDFLAGS="-fprofile-arcs -ftest-coverage" \
    --disable-doh \
    --prefix=$WORKDIR/bind-gcov/build && \
    make $MAKE_OPT && make install


COPY --chown=ubuntu:ubuntu in-dns ${WORKDIR}/in-dns
COPY --chown=ubuntu:ubuntu dns.dict ${WORKDIR}/dns.dict
COPY --chown=ubuntu:ubuntu cov_script.sh ${WORKDIR}/cov_script
COPY --chown=ubuntu:ubuntu run.sh ${WORKDIR}/run
COPY --chown=ubuntu:ubuntu named.conf ${WORKDIR}/named.conf
COPY --chown=ubuntu:ubuntu named.conf.options ${WORKDIR}/named.conf.options

RUN mkdir /etc/bind && \
    cp ${WORKDIR}/named.conf /usr/local/etc/. && \
    cp ${WORKDIR}/named.conf.options /etc/bind/.