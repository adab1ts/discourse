# NAME:     adab1ts/discourse
# VERSION:  1.0.0-dev

FROM buildpack-deps:xenial

LABEL \
  org.adabits.discourse.description="Docker-based development environment for Discourse" \
  org.adabits.discourse.vendor="Adab1ts" \
  org.adabits.discourse.version="1.0.0-dev"

MAINTAINER Carles Muiños "carles@adabits.org"

# Base
## add our user and group first to make sure their IDs get assigned consistently, regardless of whatever dependencies get added
RUN \
  groupadd --gid 1000 discourse \
  && useradd --uid 1000 --gid discourse --shell /bin/bash --create-home discourse

RUN \
  apt-get update && apt-get install -y --no-install-recommends \
  advancecomp \
  build-essential \
  debconf-utils \
  expect \
  gifsicle \
  gnupg2 \
  gsfonts \
  jhead \
  jpegoptim \
  libjpeg-turbo-progs \
  optipng \
  pngcrush \
  pngquant \
  postgresql-client \
  python-software-properties \
&& rm -rf /var/lib/apt/lists/*

# Ruby
ENV RUBY_VERSION 2.3.1
ENV RUBY_HOME /usr/local/ruby
RUN \
  set -ex \
  && mkdir -p /usr/local/etc \
  && { \
      echo 'install: --no-document'; \
      echo 'update: --no-document'; \
     } >> /usr/local/etc/gemrc \
  && RUBY_SRC="$(mktemp -d)" \
  && git clone https://github.com/rbenv/ruby-build.git $RUBY_SRC \
  && $RUBY_SRC/install.sh \
  && rm -rf $RUBY_SRC \
  && mkdir -p $RUBY_HOME \
  && ruby-build $RUBY_VERSION $RUBY_HOME/$RUBY_VERSION \
  && ln -s $RUBY_HOME/$RUBY_VERSION $RUBY_HOME/default

ENV PATH $RUBY_HOME/default/bin:$PATH
RUN \
  gem install --no-document \
  bundler \
  mailcatcher

# Node.js
ENV NODE_VERSION 6.9.2
ENV NVM_DIR /usr/local/nvm
RUN \
  set -e \
  && curl -o- https://raw.githubusercontent.com/creationix/nvm/v0.32.1/install.sh | bash \
  && . $NVM_DIR/nvm.sh \
  && nvm install $NODE_VERSION \
  && nvm alias default $NODE_VERSION \
  && nvm use default

ENV NODE_PATH $NVM_DIR/versions/node/v$NODE_VERSION/lib/node_modules
ENV PATH $NVM_DIR/versions/node/v$NODE_VERSION/bin:$PATH
RUN \
  npm install -g \
  svgo \
  phantomjs-prebuilt

# Discourse
ARG APP_HOME
ENV APP_HOME ${APP_HOME:-/app}

WORKDIR $APP_HOME

COPY Gemfile Gemfile.lock ./
RUN bundle install --jobs 5 --retry 3
COPY . ./

RUN chown -R discourse:discourse ./
USER discourse

EXPOSE 3000

CMD ["/bin/bash"]
