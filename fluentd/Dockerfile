FROM fluent/fluentd:edge-debian

USER root

# 대화형 입력을 비활성화
ENV DEBIAN_FRONTEND=noninteractive

# 필요한 플러그인 설치
RUN apt-get update && apt-get install -y \
    default-libmysqlclient-dev \
    build-essential \
    && apt-get clean \
    && rm -rf /var/lib/apt/lists/*

RUN gem install fluent-plugin-sql --no-document && \
    gem install mysql2 --no-document

RUN apt-get update -y && \
    apt-get install -y \
    ubuntu-dev-tools \
    gcc \
    default-libmysqlclient-dev && \
    gem install fluent-plugin-mysql && \
    fluent-gem install fluent-plugin-rewrite-tag-filter && \
    gem sources --clear-all && \
    rm -rf /var/lib/apt/lists/* && \
    rm -rf /tmp/* /var/tmp/* /usr/lib/ruby/gems/*/cache/*.gem

# Fluentd 설정 파일 복사
COPY ./fluent.conf /fluentd/etc/fluent.conf
COPY ./db.conf /fluentd/etc/db.conf

# 포트 노출
EXPOSE 24224

# Fluentd 실행 명령
CMD ["fluentd", "-c", "/fluentd/etc/fluent.conf"]