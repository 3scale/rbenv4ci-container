FROM alpine:3.9
MAINTAINER Alejandro Martinez Ruiz <amr@redhat.com>

ARG BASE_TOOLS="git openssh-client tar gzip ca-certificates"
# install basic build tools
RUN apk update \
    && apk upgrade \
    && apk add bash sudo git make gcc autoconf automake libtool gettext file \
       libc-dev linux-headers ${BASE_TOOLS}

SHELL ["/bin/bash", "-c"]

ARG USER_NAME=ruby
RUN addgroup ruby \
    && adduser -D -h "/home/${USER_NAME}" -s /bin/bash -G ${USER_NAME} ${USER_NAME} \
    && echo "${USER_NAME} ALL=(ALL:ALL) NOPASSWD: ALL" > \
       "/etc/sudoers.d/${USER_NAME}" \
    && chmod 0440 "/etc/sudoers.d/${USER_NAME}"

USER "${USER_NAME}"

ARG RBENV_ROOT="/home/${USER_NAME}/.rbenv"

# not really meant to be changed
ENV RBENV_ROOT="${RBENV_ROOT}" \
    RBENV_BINPATH="${RBENV_ROOT}/bin" \
    RBENV_PATH="${RBENV_ROOT}/shims:${RBENV_BINPATH}" \
    RBENV_RUBYBUILD_ROOT="${RBENV_ROOT}/plugins/ruby-build"

RUN git clone --recurse-submodules --shallow-submodules --no-tags --jobs $(nproc) \
      --depth 1 --no-checkout --progress \
    https://github.com/rbenv/rbenv.git "${RBENV_ROOT}"

RUN git clone --recurse-submodules --shallow-submodules --no-tags --jobs $(nproc) \
      --depth 1 --no-checkout --progress \
    https://github.com/rbenv/ruby-build.git "${RBENV_RUBYBUILD_ROOT}"

ARG RBENV_TAG=483e7f9bdf618ad25af6cab566982e1165274d99
RUN cd "${RBENV_ROOT}" \
    && (git checkout -q -f "${RBENV_TAG}" 2> /dev/null \
    || ((git fetch -q --jobs $(nproc) --prune --depth 1 origin tag "${RBENV_TAG}" \
         || (git fetch -q --jobs $(nproc) origin "${RBENV_TAG}" \
             || git fetch -q --jobs $(nproc) origin +refs/heads/*:refs/remotes/origin/*)) \
             && (git checkout -q -f "${RBENV_TAG}" \
                 || (git fetch -q --jobs $(nproc) --unshallow \
                     && git checkout -q -f "${RBENV_TAG}")))) \
    && echo "Ignore git errors (if any) just above."

ARG RBENV_RUBYBUILD_TAG=v20190130
RUN cd "${RBENV_RUBYBUILD_ROOT}" \
    && (git checkout -q -f "${RBENV_RUBYBUILD_TAG}" 2> /dev/null \
    || ((git fetch -q --jobs $(nproc) --prune --depth 1 origin tag "${RBENV_RUBYBUILD_TAG}" \
         || (git fetch -q --jobs $(nproc) origin "${RBENV_RUBYBUILD_TAG}" \
             || git fetch -q --jobs $(nproc) origin +refs/heads/*:refs/remotes/origin/*)) \
             && (git checkout -q -f "${RBENV_RUBYBUILD_TAG}" \
                 || (git fetch -q --jobs $(nproc) --unshallow \
                     && git checkout -q -f "${RBENV_RUBYBUILD_TAG}")))) \
    && echo "Ignore git errors (if any) just above."

RUN cd "${RBENV_ROOT}" && src/configure && make -C src || true

RUN echo -n "export PATH=${RBENV_BINPATH}" >> ~/.bash_rbenv \
    && echo ':~/.local/bin:$PATH' >> ~/.bash_rbenv \
    && echo 'eval "$(rbenv init -)"' >> ~/.bash_rbenv \
    && echo 'source ~/.bash_rbenv' >> ~/.profile

ENV PATH="${RBENV_PATH}:~/.local/bin:${PATH}"

# maybe libressl-dev instead of openssl-dev?
ARG MRI_DEPS="bzip2 zlib-dev readline-dev openssl-dev yaml-dev libffi-dev gdbm-dev ncurses-dev"
RUN test "x${MRI_DEPS}" = "x" || sudo apk add ${MRI_DEPS}

RUN echo 'gem: --no-document' >> ~/.gemrc \
    && echo --color > ~/.rspec

ARG RBENV4CI_ROOT="/home/${USER_NAME}/.rbenv4ci"
RUN git clone --recurse-submodules --shallow-submodules --no-tags --jobs $(nproc) \
      --depth 1 --no-checkout --progress \
    https://github.com/3scale/rbenv4ci.git "${RBENV4CI_ROOT}/src"

ARG RBENV4CI_TAG=v0.2.1
RUN cd "${RBENV4CI_ROOT}/src" \
    && (git checkout -q -f "${RBENV4CI_TAG}" 2> /dev/null \
    || ((git fetch -q --jobs $(nproc) --prune --depth 1 origin tag "${RBENV4CI_TAG}" \
         || (git fetch -q --jobs $(nproc) origin "${RBENV4CI_TAG}" \
             || git fetch -q --jobs $(nproc) origin +refs/heads/*:refs/remotes/origin/*)) \
             && (git checkout -q -f "${RBENV4CI_TAG}" \
                 || (git fetch -q --jobs $(nproc) --unshallow \
                     && git checkout -q -f "${RBENV4CI_TAG}")))) \
    && echo "Ignore git errors (if any) just above."

RUN cd "${RBENV4CI_ROOT}/src" \
    && ./autogen.sh \
    && ./configure --prefix="/home/${USER_NAME}/.local" \
    && make install \
    && sudo apk add coreutils grep findutils

# Installing specific Ruby versions
#
# Specify all versions to be installed separated by spaces.
# Partial versions also understood, but beware of preview/dev releases, as the
# scripts will be confused by them if they are the latest available for your
# partial version.
#
#ARG RUBY_VERSIONS="2.2 2.5"
#RUN ruby_versions ${RUBY_VERSIONS}
#
# When you have a Gemfile in the working directory, you can run:
#
# $ bundle_install_rubies
#
# Note that this will pick all Gemfile* files and install gems for all currently
# installed ruby interpreters. It will also pick up any Gemfile*.lock files and
# match Bundler versions, and will also respect any .ruby-version file present
# in the directory or its parents.
#
# You can also source the provided scripts to call specific functions, such as
# `set_ruby_version`, which will also do partial version matching and change the
# current interpreter, which is useful for running your test suite with
# different versions.
