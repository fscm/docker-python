# global args
ARG __BUILD_DIR__="/build"
ARG __DATA_DIR__="/work"



FROM fscm/centos:8 as build

ARG __BUILD_DIR__
ARG __DATA_DIR__
ARG PYTHON3_VERSION="3.8.5"
ARG __USER__="root"
ARG __WORK_DIR__="/work"
ARG __SOURCE_DIR__="${__WORK_DIR__}/src"

ENV \
  LANG="C.utf8" \
  LC_ALL="C.utf8"

USER ${__USER__}

#COPY --chown="${__USER__}":${__USER__} "LICENSE" "files/" "${__WORK_DIR__}/"
COPY "LICENSE" "files/" "${__WORK_DIR__}/"

WORKDIR "${__WORK_DIR__}"

SHELL ["/bin/bash", "-c"]

RUN \
# build env
  echo '=== setting build env ===' && \
  time { \
    set +h && \
    export __NPROC__="$(getconf _NPROCESSORS_ONLN || echo 1)" && \
    export CFLAGS='-O2 -g0 -s -w -pipe -m64 -mtune=generic -fno-semantic-interposition -DNDEBUG -D_GNU_SOURCE -fPIC -fwrapv' && \
    export CFLAGS_NODIST='-O2 -g0 -s -w -pipe -m64 -mtune=generic -fno-semantic-interposition -DNDEBUG -D_GNU_SOURCE -fPIC -fwrapv' && \
    export CPPFLAGS='-I/usr/include/x86_64-linux-gnu' && \
    export LDFLAGS='-g0 -Wl,-z,relro -Wl,-z,now -Wl,--strip-all -L/usr/lib/x86_64-linux-gnu' && \
    export LDFLAGS_NODIST='-g0 -Wl,-z,relro -Wl,-z,now -Wl,--strip-all -L/usr/lib/x86_64-linux-gnu' && \
    export LINKCC='gcc' && \
    export MAKEFLAGS="--silent --output-sync --no-print-directory --jobs ${__NPROC__}" && \
    export OPT='-O2 -g0 -s -w -pipe -m64 -mtune=generic -fno-semantic-interposition -DNDEBUG -D_GNU_SOURCE -fPIC -fwrapv' && \
    export OS_CODENAME="$(. /etc/os-release; echo ${VERSION_CODENAME})" && \
    export PKG_CONFIG_ALLOW_SYSTEM_CFLAGS=1 && \
    export TIMEFORMAT='=== time taken: %lR (s:%lS u:%lU)' ; \
  } && \
# build structure
  echo '=== creating build structure ===' && \
  time { \
    for folder in '/licenses'; do install --directory --owner="${__USER__}" --group="${__USER__}" --mode=0755 "${__BUILD_DIR__}${folder}"; done && \
    for folder in '/tmp' "${__DATA_DIR__}"; do install --directory --owner="${__USER__}" --group="${__USER__}" --mode=1777 "${__BUILD_DIR__}${folder}"; done ; \
  } && \
# dependencies
  echo '=== instaling dependencies ===' && \
  time { \
    yum --assumeyes --quiet --setopt=install_weak_deps=False --enablerepo=PowerTools install \
      binutils \
      bluez-libs \
      bzip2-devel \
      ca-certificates \
      centos-repos \
      coreutils-single \
      curl \
      dbus \
      device-mapper-persistent-data \
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
      sqlite-devel \
      tar \
      tk-devel \
      wget \
      xz \
      xz-devel \
      yum \
      yum-utils \
      zlib-devel ; \
  } && \
# build base os
  echo '=== building base os ===' && \
  time { \
    yumdownloader --assumeyes --quiet --arch="$(arch)" --destdir="${__WORK_DIR__}" centos-release && \
    rpm --root "${__BUILD_DIR__}" --initdb && \
    rpm --root "${__BUILD_DIR__}" --import /etc/pki/rpm-gpg/RPM-GPG-KEY-centosofficial && \
    rpm --root "${__BUILD_DIR__}" --install --nodeps --noscripts "${__WORK_DIR__}"/centos-release*.rpm && \
    yum --assumeyes --quiet --disablerepo='*' --enablerepo='BaseOS' --enablerepo='AppStream' --installroot="${__BUILD_DIR__}" --setopt='tsflags=nodocs' --setopt='install_weak_deps=False' --setopt='override_install_langs=C.utf8' install \
      bash \
      bzip2-libs \
      ca-certificates \
      centos-repos \
      coreutils-single \
      curl \
      expat \
      fontconfig \
      freetype \
      gdbm-libs \
      glibc \
      glibc-minimal-langpack \
      keyutils-libs \
      krb5-libs \
      hostname \
      libX11 \
      libXau \
      libXft \
      libXrender \
      libcom_err \
      libffi \
      libnsl2 \
      libpng \
      libselinux \
      libtirpc \
      libuuid \
      libxcb \
      libxcrypt \
      ncurses-libs \
      openssl-libs \
      pcre2 \
      readline \
      sqlite-libs \
      tar \
      tcl \
      tk \
      xz-libs \
      yum \
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
      --with-dbmliborder=gdbm:ndbm:bdb \
      --with-ensurepip \
      --with-lto \
      --with-ssl-default-suites='openssl' \
      --with-system-expat \
      --with-system-ffi \
      --without-dtrace \
      --without-pydebug \
      --enable-ipv6 \
      --enable-loadable-sqlite-extensions \
      --enable-optimizations \
      --enable-option-checking='fatal' \
      --enable-shared && \
    make CFLAGS_NODIST="${CFLAGS_NODIST}" > /dev/null && \
    make DESTDIR="${__BUILD_DIR__}" install > /dev/null && \
    find "${__BUILD_DIR__}${_py_prefix_}/lib/python${_py_version_}" -depth -type d -a \( -name 'test' -o -name 'tests' -o -name 'idle_test' -o -name '__pycache__' \) -exec rm -rf '{}' + && \
    find "${__BUILD_DIR__}${_py_prefix_}/lib/python${_py_version_}" -depth -type f -a \( -name '*.pyc' -o -name '*.pyo' -o -name '*.bat' -o -name '*LICENSE*' \) -delete && \
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
    sed -i -e '/best=True/a exclude=*.i?86\ntsflags=nodocs' "${__BUILD_DIR__}/etc/yum.conf" && \
    echo '' > "${__BUILD_DIR__}/etc/resolv.conf" && \
    echo 'host' > "${__BUILD_DIR__}/etc/hostname" && \
    echo 'container' > "${__BUILD_DIR__}/etc/dnf/vars/infra" && \
    #echo 'docker' > "${__BUILD_DIR__}/run/systemd/container" && \
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
    find "${__BUILD_DIR__}/usr/share/doc" -mindepth 1 -not -type d -not \( -iname 'COPYING*' -o -iname 'GPL' -o -iname '*LICENSE*' -o -iname 'copyright' \) -delete && \
    find "${__BUILD_DIR__}/usr/share/doc" -mindepth 1 -type d -empty -delete && \
    find "${__BUILD_DIR__}/var/cache" -type f -delete && \
    find "${__BUILD_DIR__}/etc/yum.repos.d" -not \( -iname '*base*' -o -iname '*appstream*' -o -iname '*extras*' -o -iname '*powertools*' -o -iname '*centosplus*' \) -type f -delete ; \
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
ARG __DATA_DIR__

LABEL \
  maintainer="Frederico Martins <https://hub.docker.com/u/fscm/>" \
  vendor="fscm" \
  cmd="docker container run --interactive --rm --tty fscm/python:centos bash" \
  params="--volume ./:${__DATA_DIR__}:rw"

ENV \
  LANG="C.utf8" \
  LC_ALL="C.utf8"

COPY --from=build "${__BUILD_DIR__}" "/"

VOLUME ["${__DATA_DIR__}"]

WORKDIR "${__DATA_DIR__}"

CMD ["/bin/bash"]
