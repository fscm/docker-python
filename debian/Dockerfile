# global args
ARG __BUILD_DIR__="/build"
ARG __WORK_DIR__="/work"



FROM fscm/debian:buster as build

ARG __BUILD_DIR__
ARG __WORK_DIR__
ARG PYTHON3_VERSION="3.9.5"
ARG __USER__="root"
ARG __SOURCE_DIR__="${__WORK_DIR__}/src"

ENV \
  LANG="C.UTF-8" \
  LC_ALL="C.UTF-8" \
  DEBCONF_NONINTERACTIVE_SEEN="true" \
  DEBIAN_FRONTEND="noninteractive"

USER ${__USER__}

#COPY --chown="${__USER__}":${__USER__} "LICENSE" "files/" "${__WORK_DIR__}/"
COPY "LICENSE" "files/" "${__WORK_DIR__}"/

WORKDIR "${__WORK_DIR__}"

SHELL ["/bin/bash", "-c"]

RUN \
# build env
  echo '=== setting build env ===' && \
  time { \
    set +h && \
    export __ARCH__="$(arch | sed 's/_/-/g')" && \
    export __NPROC__="$(getconf _NPROCESSORS_ONLN || echo 1)" && \
    export CFLAGS="-O2 -g0 -s -w -pipe -fno-semantic-interposition -march=${__ARCH__} -mtune=generic -DNDEBUG -D_GNU_SOURCE" && \
    export LDFLAGS='-g0 -fno-semantic-interposition -Wl,-z,relro -Wl,-z,now -Wl,--strip-all' && \
    export MAKEFLAGS="--silent --no-print-directory --jobs ${__NPROC__}" && \
    export OS_CODENAME="$(. /etc/os-release; echo ${VERSION_CODENAME})" && \
    export PKG_CONFIG_ALLOW_SYSTEM_CFLAGS=1 && \
    export TIMEFORMAT='=== time taken: %lR (s:%lS u:%lU)' ; \
  } && \
# build structure
  echo '=== creating build structure ===' && \
  time { \
    for folder in '/licenses'; do install --directory --owner="${__USER__}" --group="${__USER__}" --mode=0755 "${__BUILD_DIR__}${folder}"; done ; \
  } && \
# dependencies
  echo '=== instaling dependencies ===' && \
  time { \
    apt-get -qq update && \
    apt-get -qq -y -o=Dpkg::Use-Pty=0 --no-install-recommends install \
      binutils \
      ca-certificates \
      coreutils \
      curl \
      dbus \
      debian-archive-keyring \
      debootstrap \
      dpkg-dev \
      findutils \
      g++ \
      gcc \
      gzip \
      libbluetooth-dev \
      libbz2-dev \
      libc6-dev \
      libexpat1-dev \
      libffi6 \
      libffi-dev \
      libgdbm-dev \
      liblzma-dev \
      libncurses-dev \
      libreadline-dev \
      libsqlite3-dev \
      libssl-dev \
      make \
      netbase \
      pkg-config \
      tar \
      tk-dev \
      tzdata \
      uuid-dev \
      wget \
      xz-utils \
      zlib1g-dev \
      > /dev/null 2>&1 ; \
  } && \
# build base os
  echo '=== building base os ===' && \
  time { \
    debootstrap \
      --variant='minbase' \
      --include=ca-certificates,libbz2-1.0,libc6,libexpat1,libffi6,libgdbm6,liblzma5,libncursesw6,libreadline7,libsqlite3-0,libssl1.1,libtinfo6,libuuid1,netbase,tzdata,zlib1g \
      "${OS_CODENAME}" \
      "${__BUILD_DIR__}" && \
    install --directory --owner="${__USER__}" --group="${__USER__}" --mode=0755 "${__BUILD_DIR__}/licenses/debian" && \
    install --owner="${__USER__}" --group="${__USER__}" --mode=0644 --target-directory="${__BUILD_DIR__}/licenses/debian" /licenses/debian/social-contract.txt ; \
  } && \
# copy tests
  echo '=== copying test files ===' && \
  time { \
    install --owner="${__USER__}" --group="${__USER__}" --mode=0755 --target-directory="${__BUILD_DIR__}/usr/bin" "${__WORK_DIR__}/tests"/* ; \
  } && \
# python3
  echo '=== installing python3 ===' && \
  time { \
    install --directory "${__SOURCE_DIR__}/python3/_build" && \
    curl --silent --location --retry 3 "https://www.python.org/ftp/python/${PYTHON3_VERSION%%[a-z]*}/Python-${PYTHON3_VERSION}.tar.xz" \
      | tar xJ --no-same-owner --strip-components=1 -C "${__SOURCE_DIR__}/python3" && \
    cd "${__SOURCE_DIR__}/python3/_build" && \
    sed -i.orig -e '/^Cflags:/ s|:.*|: -I${includedir}/x86_64-linux-gnu|' /usr/lib/x86_64-linux-gnu/pkgconfig/libffi.pc && \
    rm -rf "../Modules/expat" && \
    _py_prefix_='/usr/local' && \
    _py_version_="$(echo ${PYTHON3_VERSION} | sed -n 's/^\([0-9]*\.[0-9]*\).*/\1/p')" && \
    ../configure \
      --quiet \
      --prefix="${_py_prefix_}" \
      --exec_prefix="${_py_prefix_}" \
      --libexecdir="${_py_prefix_}/lib" \
      --with-computed-gotos \
      --with-ensurepip='install' \
      --with-lto \
      --with-ssl-default-suites='openssl' \
      --with-system-expat \
      --with-system-ffi \
      --without-dtrace \
      --without-pydebug \
      --enable-ipv6 \
      --enable-loadable-sqlite-extensions \
      --enable-optimizations \
      --enable-option-checking='fatal' && \
    make > /dev/null && \
    make DESTDIR="${__BUILD_DIR__}" install > /dev/null && \
    find "${__BUILD_DIR__}${_py_prefix_}/lib" -depth -type d -a \( -name 'test' -o -name 'tests' -o -name 'idle_test' -o -name '__pycache__' \) -exec rm -rf '{}' + && \
    find "${__BUILD_DIR__}${_py_prefix_}/lib" -depth -type f -a \( -name '*.pyc' -o -name '*.pyo' -o -name '*.bat' -o -name '*.a' -o -name '*LICENSE*' \) -delete && \
    for p in $(find "${__BUILD_DIR__}${_py_prefix_}/bin" -type f -name "*${_py_version_}*"); do \
      s="$(basename ${p})"; \
      t="${__BUILD_DIR__}${_py_prefix_}/bin/$(echo ${s} | sed -e "s/\(\|-\)${_py_version_}//")"; \
      if [ ! -f "${t}" ]; then ln -s "${s}" "${t}"; fi; \
    done && \
    unset _py_version_ && \
    unset _py_prefix_ && \
    echo '/usr/local/lib' > "${__BUILD_DIR__}/etc/ld.so.conf.d/usrlocal.conf" && \
    chroot "${__BUILD_DIR__}" '/usr/sbin/ldconfig' && \
    install --directory --owner="${__USER__}" --group="${__USER__}" --mode=0755 "${__BUILD_DIR__}/licenses/python3" && \
    (cd .. && find ./ -type f -a \( -iname '*LICENSE*' -o -iname '*COPYING*' \) -exec cp --parents {} "${__BUILD_DIR__}/licenses/python3" ';') && \
    cd ~- && \
    rm -rf "${__SOURCE_DIR__}/python3" ; \
  } && \
# stripping
  echo '=== stripping libraries and binaries ===' && \
  time { \
    for folder in lib lib64 local/lib; do \
      find "${__BUILD_DIR__}/usr/${folder}" -type f -name '*.a' -exec strip --strip-debug {} ';' ; \
      find "${__BUILD_DIR__}/usr/${folder}" -type f -name '*.so*' -exec strip --strip-unneeded {} ';' ; \
    done && \
    for folder in bin sbin local/bin local/sbin; do \
      find "${__BUILD_DIR__}/usr/${folder}" -type f -not -links +1 -exec strip --strip-all {} ';' ; \
    done ; \
  } && \
# system settings / cleanup
  echo '=== system settings / cleaning up ===' && \
  time { \
    echo "deb http://deb.debian.org/debian ${OS_CODENAME} main non-free contrib" > "${__BUILD_DIR__}/etc/apt/sources.list" && \
    echo "deb http://security.debian.org/ ${OS_CODENAME}/updates main contrib non-free" >> "${__BUILD_DIR__}/etc/apt/sources.list" && \
    echo "deb http://deb.debian.org/debian ${OS_CODENAME}-updates main contrib non-free" >> "${__BUILD_DIR__}/etc/apt/sources.list" && \
    rm -f "${__BUILD_DIR__}/etc/apt/apt.conf.d/01autoremove-kernels" && \
    APT_GET_CLEAN='"rm -f /var/cache/apt/archives/*.deb /var/cache/apt/archives/partial/*.deb /var/cache/apt/*.bin || true";' && \
    printf 'DPkg::Post-Invoke { %s };\nAPT::Update::Post-Invoke { %s };\nDir::Cache::pkgcache "";\nDir::Cache::srcpkgcache "";\n' "${APT_GET_CLEAN}" "${APT_GET_CLEAN}" > "${__BUILD_DIR__}/etc/apt/apt.conf.d/90docker-clean" && \
    printf 'DPkg::Use-Pty "false";\n' > "${__BUILD_DIR__}/etc/apt/apt.conf.d/90docker-no-dpkg-output" && \
    printf 'Acquire::Languages "none";\n' > "${__BUILD_DIR__}/etc/apt/apt.conf.d/90docker-no-languages" && \
    printf 'Acquire::GzipIndexes "true";\nAcquire::CompressionTypes::Order:: "gz";\n' > "${__BUILD_DIR__}/etc/apt/apt.conf.d/90docker-gzip-indexes" && \
    printf 'APT::AutoRemove::SuggestsImportant "false";\n' > "${__BUILD_DIR__}/etc/apt/apt.conf.d/90docker-autoremove-suggests" && \
    printf 'APT::Install-Recommends "false";\n' > "${__BUILD_DIR__}/etc/apt/apt.conf.d/90docker-no-recommends" && \
    printf 'force-unsafe-io\n' > "${__BUILD_DIR__}/etc/dpkg/dpkg.cfg.d/90docker-apt-speedup" && \
    printf 'path-exclude=/usr/share/doc/*\npath-exclude=/usr/share/info/*\npath-exclude=/usr/share/locale/*\npath-exclude=/usr/share/man/*\npath-include=/usr/share/doc/*/copyright\n' > "${__BUILD_DIR__}/etc/dpkg/dpkg.cfg.d/90docker-excludes" && \
    echo '' > "${__BUILD_DIR__}/etc/resolv.conf" && \
    echo 'host' > "${__BUILD_DIR__}/etc/hostname" && \
    :> "${__BUILD_DIR__}/etc/machine-id" && \
    find "${__BUILD_DIR__}/etc" -path '*/systemd/system/*.wants/*' \( -type l -o -type f \) -delete && \
    if [ -x "${__BUILD_DIR__}/usr/bin/dpkg-divert" ]; then chroot "${__BUILD_DIR__}/" dpkg-divert --local --rename --add /sbin/initctl; fi && \
    printf '#!/bin/sh\nexit 101\n' > "${__BUILD_DIR__}/usr/sbin/policy-rc.d" && \
    printf '#!/bin/sh\nexit 0\n' > "${__BUILD_DIR__}/sbin/initctl" && \
    chmod 0755 "${__BUILD_DIR__}/usr/sbin/policy-rc.d" "${__BUILD_DIR__}/sbin/initctl" && \
    rm -rf "${__BUILD_DIR__}/usr/local/share/info"/* && \
    rm -rf "${__BUILD_DIR__}/usr/local/share/man"/* && \
    rm -rf "${__BUILD_DIR__}/usr/share/info"/* && \
    rm -rf "${__BUILD_DIR__}/usr/share/man"/* && \
    rm -rf "${__BUILD_DIR__}/var/cache/apt"/* && \
    rm -rf "${__BUILD_DIR__}/var/lib/apt/lists"/* && \
    rm -rf "${__BUILD_DIR__}/var/log"/* && \
    rm -rf "${__BUILD_DIR__}/dev"/.??* && \
    rm -rf "${__BUILD_DIR__}/home"/.??* && \
    rm -rf "${__BUILD_DIR__}/root"/.??* && \
    rm -rf "${__BUILD_DIR__}/tmp"/.??* && \
    rm -rf "${__BUILD_DIR__}/boot" && \
    rm -rf "${__BUILD_DIR__}/proc" && \
    rm -rf "${__BUILD_DIR__}/sys" && \
    find "${__BUILD_DIR__}/usr/lib/x86_64-linux-gnu/gconv" -mindepth 1 -maxdepth 1 -type f -not -name 'UTF*' -delete && \
    find "${__BUILD_DIR__}/usr/share/locale" -mindepth 1 -maxdepth 1 -type d -not \( -name 'en' -o -name 'en_US' \) -exec rm -r {} + && \
    find "${__BUILD_DIR__}/usr/share/doc" -mindepth 1 -not -type d -not \( -iname 'COPYING*' -o -iname 'GPL' -o -iname '*LICENSE*' -o -iname 'copyright' \) -delete && \
    find "${__BUILD_DIR__}/usr/share/doc" -mindepth 1 -type d -empty -delete && \
    find "${__BUILD_DIR__}/var/cache" -type f -delete ; \
  } && \
# licenses
  echo '=== project licenses ===' && \
  time { \
    install --owner=${__USER__} --group=${__USER__} --mode=0644 --target-directory="${__BUILD_DIR__}/licenses" "${__WORK_DIR__}/LICENSE" ; \
  } && \
# done
  echo '=== all done! ==='



FROM scratch

ARG __BUILD_DIR__
ARG __WORK_DIR__

LABEL \
  maintainer="Frederico Martins <https://hub.docker.com/u/fscm/>" \
  vendor="fscm" \
  cmd="docker container run --interactive --rm --tty fscm/python:debian" \
  params="--volume ./:${__WORK_DIR__}:rw"

ENV \
  LANG="C.UTF-8" \
  LC_ALL="C.UTF-8" \
  DEBCONF_NONINTERACTIVE_SEEN="true" \
  DEBIAN_FRONTEND="noninteractive"

COPY --from=build "${__BUILD_DIR__}" "/"

VOLUME ["${__WORK_DIR__}"]

WORKDIR "${__WORK_DIR__}"

CMD ["/bin/bash"]
