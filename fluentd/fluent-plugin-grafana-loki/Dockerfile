FROM fluent/fluentd:v1.7.2-debian-1.0

USER root
WORKDIR /home/fluent
ENV PATH /fluentd/vendor/bundle/ruby/2.6.0/bin:$PATH
ENV GEM_PATH /fluentd/vendor/bundle/ruby/2.6.0
ENV GEM_HOME /fluentd/vendor/bundle/ruby/2.6.0
# skip runtime bundler installation
ENV FLUENTD_DISABLE_BUNDLER_INJECTION 1

COPY docker/Gemfile* /fluentd/
RUN buildDeps="sudo make gcc g++ libc-dev" \
       && apt-get update \
       && apt-get install -y --no-install-recommends \
       $buildDeps libsystemd0 net-tools \
       && gem install bundler --version 2.0.2 \
       && bundle config silence_root_warning true \
       && bundle install --gemfile=/fluentd/Gemfile --path=/fluentd/vendor/bundle \
       && sudo gem sources --clear-all \
       && SUDO_FORCE_REMOVE=yes \
       apt-get purge -y --auto-remove \
       -o APT::AutoRemove::RecommendsImportant=false \
       $buildDeps \
       && rm -rf /var/lib/apt/lists/* \
       /home/fluent/.gem/ruby/2.6.0/cache/*.gem \
       /tmp/* /var/tmp/* /usr/lib/ruby/gems/*/cache/*.gem

COPY docker/entrypoint.sh /fluentd/entrypoint.sh
COPY lib/fluent/plugin/out_loki.rb /fluentd/plugins/out_loki.rb
COPY docker/conf/ /fluentd/etc/loki/

ENV FLUENTD_CONF="/fluentd/etc/loki/fluentd.conf"
ENV FLUENTD_OPT=""

ENV LOKI_URL "http://127.0.0.1:13100"
ENV LOKI_ENV "dev"

# Overwrite ENTRYPOINT to run fluentd as root for /var/log / /var/lib
ENTRYPOINT ["/fluentd/entrypoint.sh"]