FROM centos:7
MAINTAINER Alejandro Martinez Ruiz <amr@redhat.com>

ARG BASE_TOOLS="git openssh-clients tar gzip ca-certificates"
# set localtime to UTC and install basic build tools
RUN (ln -s /usr/share/zoneinfo/UTC /etc/localtime 2> /dev/null || true) \
    && sed -i /etc/yum.conf -e \
    '/^\(clean_requirements_on_remove\|history_record\|tsflags\|logfile\)=/d' -e \
    's/^\(\[main\]\b.*\)$/\1\nclean_requirements_on_remove=1\nhistory_record=0\ntsflags=nodocs\nlogfile=\/dev\/null\n/' \
      /etc/yum.conf \
    && yum upgrade -y \
    && yum distro-sync -y \
    && yum install -y sudo git make gcc autoconf automake libtool gettext file \
       ${BASE_TOOLS} \
    && yum autoremove -y \
    && yum -y clean all \
    && rm -rf /var/cache/yum

ARG USER_NAME=ruby
RUN useradd -m -d "/home/${USER_NAME}" -l -G wheel ${USER_NAME} \
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

# update to git clone to include --shallow-submodules --no-tags --jobs $(nproc)
# when newer git versions land in the base distro
RUN git clone --recurse-submodules --depth 1 --no-checkout --progress \
    https://github.com/rbenv/rbenv.git "${RBENV_ROOT}"

RUN git clone --recurse-submodules --depth 1 --no-checkout --progress \
    https://github.com/rbenv/ruby-build.git "${RBENV_RUBYBUILD_ROOT}"

ARG RBENV_TAG=c8ba27fd07e4bf3c444df301e5fb2a5fcdacaf9d
# update git fetch to include --jobs $(nproc) when newer git versions land
RUN cd "${RBENV_ROOT}" \
    && (git checkout -q -f "${RBENV_TAG}" 2> /dev/null \
    || ((git fetch -q --prune --depth 1 origin tag "${RBENV_TAG}" \
         || (git fetch -q origin "${RBENV_TAG}" \
             || git fetch -q origin +refs/heads/*:refs/remotes/origin/*)) \
         && git checkout -q -f "${RBENV_TAG}")) \
    && echo "Ignore git errors (if any) just above."

ARG RBENV_RUBYBUILD_TAG=v20180424
RUN cd "${RBENV_RUBYBUILD_ROOT}" \
    && (git checkout -q -f "${RBENV_RUBYBUILD_TAG}" 2> /dev/null \
    || ((git fetch -q --prune --depth 1 origin tag "${RBENV_RUBYBUILD_TAG}" \
         || (git fetch -q origin "${RBENV_RUBYBUILD_TAG}" \
             || git fetch -q origin +refs/heads/*:refs/remotes/origin/*)) \
         && git checkout -q -f "${RBENV_RUBYBUILD_TAG}")) \
    && echo "Ignore git errors (if any) just above."

RUN sudo yum update -y
RUN cd "${RBENV_ROOT}" && src/configure && make -C src || true

RUN echo -n "export PATH=${RBENV_BINPATH}" >> ~/.bash_rbenv \
    && echo ':~/.local/bin:$PATH' >> ~/.bash_rbenv \
    && echo 'eval "$(rbenv init -)"' >> ~/.bash_rbenv \
    && echo 'source ~/.bash_rbenv' >> ~/.profile

ENV PATH="${RBENV_PATH}:~/.local/bin:${PATH}"

ARG MRI_DEPS="bzip2 zlib-devel readline-devel openssl-devel libyaml-devel libffi-devel gdbm-devel ncurses-devel"
RUN test "x${MRI_DEPS}" = "x" || sudo yum install -y ${MRI_DEPS}

RUN echo 'gem: --no-document' >> ~/.gemrc \
    && echo --color > ~/.rspec

ARG RBENV4CI_ROOT="/home/${USER_NAME}/.rbenv4ci"
RUN git clone --recurse-submodules --depth 1 --no-checkout --progress \
    https://github.com/unleashed/rbenv4ci.git "${RBENV4CI_ROOT}/src"

ARG RBENV4CI_TAG=v0.2.1
RUN cd "${RBENV4CI_ROOT}/src" \
    && (git checkout -q -f "${RBENV4CI_TAG}" 2> /dev/null \
    || ((git fetch -q --prune --depth 1 origin tag "${RBENV4CI_TAG}" \
         || (git fetch -q origin "${RBENV4CI_TAG}" \
             || git fetch -q origin +refs/heads/*:refs/remotes/origin/*)) \
         && git checkout -q -f "${RBENV4CI_TAG}")) \
    && echo "Ignore git errors (if any) just above."

RUN cd "${RBENV4CI_ROOT}/src" \
    && ./autogen.sh \
    && ./configure --prefix="/home/${USER_NAME}/.local" \
    && make install

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
