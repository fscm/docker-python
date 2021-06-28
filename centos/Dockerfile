# global args
ARG __BUILD_DIR__="/build"
ARG __WORK_DIR__="/work"



FROM fscm/centos:stream-8 as build

ARG __BUILD_DIR__
ARG __WORK_DIR__
ARG PYTHON3_VERSION="3.9.5"
ARG __USER__="root"
ARG __SOURCE_DIR__="${__WORK_DIR__}/src"

ENV \
  LANG="C.UTF-8" \
  LC_ALL="C.UTF-8"

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
    dnf --assumeyes --quiet --setopt=install_weak_deps='no' --enablerepo='powertools' install \
      binutils \
      bluez-libs \
      bzip2-devel \
      ca-certificates \
      coreutils-single \
      curl \
      dbus \
      device-mapper-persistent-data \
      dnf \
      dnf-plugins-core \
      expat-devel \
      findutils \
      gcc \
      gcc-c++ \
      gdbm-devel \
      gzip \
      libffi \
      libffi-devel \
      libnsl2-devel \
      libtirpc-devel \
      libuuid-devel \
      lvm2 \
      make \
      ncurses-devel \
      openssl-devel \
      pkg-config \
      readline-devel \
      setup \
      sqlite-devel \
      tar \
      tk-devel \
      tzdata \
      wget \
      xz \
      xz-devel \
      zlib-devel ; \
  } && \
# build base os
  echo '=== building base os ===' && \
  time { \
    dnf download --assumeyes --quiet --arch="$(arch),noarch" --destdir="${__WORK_DIR__}" centos-stream-release && \
    rpm --root "${__BUILD_DIR__}" --initdb && \
    rpm --root "${__BUILD_DIR__}" --import /etc/pki/rpm-gpg/RPM-GPG-KEY-centosofficial && \
    rpm --root "${__BUILD_DIR__}" --install --nodeps --noscripts "${__WORK_DIR__}"/centos-stream-release*.rpm && \
    dnf --assumeyes --quiet --disablerepo='*' --enablerepo='baseos' --enablerepo='appstream' --installroot="${__BUILD_DIR__}" --setopt=tsflags='nodocs' --setopt=install_weak_deps='no' install \
      bash \
      bzip2-libs \
      ca-certificates \
      centos-*repos \
      coreutils-single \
      dnf \
      expat \
      gdbm-libs \
      glibc \
      glibc-minimal-langpack \
      hostname \
      keyutils-libs \
      krb5-libs \
      libcom_err \
      libffi \
      libnsl2 \
      libselinux \
      libtirpc \
      libuuid \
      libxcrypt \
      ncurses-libs \
      openssl-libs \
      pcre2 \
      readline \
      setup \
      sqlite-libs \
      tar \
      tzdata \
      xz-libs \
      zlib && \
    install --directory --owner="${__USER__}" --group="${__USER__}" --mode=0755 "${__BUILD_DIR__}/licenses/centos" && \
    ln --symbolic --force '/usr/share/centos-release/EULA' "${__BUILD_DIR__}/licenses/centos/EULA" ; \
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
    find "${__BUILD_DIR__}/lib/python"* "${__BUILD_DIR__}/lib64/python"* -depth -type d -a \( -name 'test' -o -name 'tests' -o -name 'idle_test' -o -name '__pycache__' \) -exec rm -rf '{}' + && \
    find "${__BUILD_DIR__}/lib/python"* "${__BUILD_DIR__}/lib64/python"* -depth -type f -a \( -name '*.pyc' -o -name '*.pyo' -o -name '*.bat' -o -name '*.a' -o -name '*LICENSE*' \) -delete && \
    sed -i -e '/best=True/a exclude=*.i?86\ntsflags=nodocs' "${__BUILD_DIR__}/etc/dnf/dnf.conf" && \
    echo 'container' > "${__BUILD_DIR__}/etc/dnf/vars/infra" && \
    echo '%_install_langs C.utf8' > "${__BUILD_DIR__}/etc/rpm/macros.image-language-conf" && \
    :> "${__BUILD_DIR__}/etc/machine-id" && \
    find "${__BUILD_DIR__}/lib/systemd/system" -path '*/sysinit.target.wants/*' \( -type l -o -type f \) -not -name 'systemd-tmpfiles-setup.service' -delete && \
    find "${__BUILD_DIR__}/etc" -path '*/systemd/system/*.wants/*' \( -type l -o -type f \) -delete && \
    rm -f "${__BUILD_DIR__}/lib/systemd/system/anaconda.target.wants"/* && \
    rm -f "${__BUILD_DIR__}/lib/systemd/system/basic.target.wants"/* && \
    rm -f "${__BUILD_DIR__}/lib/systemd/system/local-fs.target.wants"/* && \
    rm -f "${__BUILD_DIR__}/lib/systemd/system/multi-user.target.wants"/* && \
    rm -f "${__BUILD_DIR__}/lib/systemd/system/sockets.target.wants"/*udev* && \
    rm -f "${__BUILD_DIR__}/lib/systemd/system/sockets.target.wants"/*initctl* && \
    if [ -x "${__BUILD_DIR__}/usr/bin/systemctl" ]; then chroot "${__BUILD_DIR__}/" systemctl mask systemd-logind.service getty.target console-getty.service sys-fs-fuse-connections.mount systemd-remount-fs.service dev-hugepages.mount; fi && \
    rm -rf "${__BUILD_DIR__}/usr/local/share/info"/* && \
    rm -rf "${__BUILD_DIR__}/usr/local/share/man"/* && \
    rm -rf "${__BUILD_DIR__}/usr/share/info"/* && \
    rm -rf "${__BUILD_DIR__}/usr/share/man"/* && \
    rm -rf "${__BUILD_DIR__}/var/cache/yum"/* && \
    rm -rf "${__BUILD_DIR__}/var/lib/dnf"/history.* && \
    rm -rf "${__BUILD_DIR__}/var/log"/* && \
    rm -rf "${__BUILD_DIR__}/sbin/sln" && \
    rm -rf "${__BUILD_DIR__}/dev"/.??* && \
    rm -rf "${__BUILD_DIR__}/home"/.??* && \
    rm -rf "${__BUILD_DIR__}/root"/.??* && \
    rm -rf "${__BUILD_DIR__}/tmp"/.??* && \
    rm -rf "${__BUILD_DIR__}/boot" && \
    rm -rf "${__BUILD_DIR__}/proc" && \
    rm -rf "${__BUILD_DIR__}/sys" && \
    find "${__BUILD_DIR__}/usr/share/i18n/locales" -mindepth 1 -maxdepth 1 -not -name 'en_US' -exec rm -r {} + && \
    find "${__BUILD_DIR__}/usr/lib64/gconv" -mindepth 1 -maxdepth 1 -type f -not -name 'UTF*' -delete && \
    find "${__BUILD_DIR__}/usr/share/locale" -mindepth 1 -maxdepth 1 -type d -not \( -name 'en' -o -name 'en_US' \) -exec rm -r {} + && \
    find "${__BUILD_DIR__}/usr/share/doc" -mindepth 1 -not -type d -not \( -name 'COPYING*' -o -name 'GPL' -o -name '*LICENSE*' \) -delete && \
    find "${__BUILD_DIR__}/usr/share/doc" -mindepth 1 -type d -empty -delete && \
    find "${__BUILD_DIR__}/var/cache" -type f -delete && \
    find "${__BUILD_DIR__}/etc/yum.repos.d" \( -iname '*debuginfo*' -o -iname '*media*' \) -type f -delete ; \
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
  cmd="docker container run --interactive --rm --tty fscm/python:centos" \
  params="--volume ./:${__WORK_DIR__}:rw"

ENV \
  LANG="C.utf8" \
  LC_ALL="C.utf8"

COPY --from=build "${__BUILD_DIR__}" "/"

VOLUME ["${__WORK_DIR__}"]

WORKDIR "${__WORK_DIR__}"

CMD ["/bin/bash"]
