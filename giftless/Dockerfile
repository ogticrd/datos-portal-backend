FROM datopian/giftless

ENV GIFTLESS_CONFIG_FILE=/etc/giftless.yaml

# Set variables from build arguments
ARG PORT=5000
EXPOSE $PORT

ARG GCP_PROJECT
ENV GCP_PROJECT ${GCP_PROJECT}

ARG GCP_BUCKET_NAME
ENV GCP_BUCKET_NAME ${GCP_BUCKET_NAME}

ARG GCP_BUCKET_KEY
ENV GCP_BUCKET_KEY ${GCP_BUCKET_KEY}

ARG CKAN_SITE_URL
ENV CKAN_SITE_URL ${CKAN_SITE_URL}

# Use transformed enviroment variables from Giftless
ENV GIFTLESS_CONFIG_TRANSFER_ADAPTERS_BASIC_OPTIONS_STORAGE_OPTIONS_PROJECT_NAME ${GCP_PROJECT}
ENV GIFTLESS_CONFIG_TRANSFER_ADAPTERS_BASIC_OPTIONS_STORAGE_OPTIONS_BUCKET_NAME ${GCP_BUCKET_NAME}
ENV GIFTLESS_CONFIG_TRANSFER_ADAPTERS_BASIC_OPTIONS_STORAGE_OPTIONS_ACCOUNT_KEY_BASE64 ${GCP_BUCKET_KEY}

COPY giftless.yaml ${GIFTLESS_CONFIG_FILE}

# Install envsubst
ENV BUILD_DEPS="gettext"  \
    RUNTIME_DEPS="libintl"

RUN set -x && \
    apk add --update $RUNTIME_DEPS && \
    apk add --virtual build_deps $BUILD_DEPS &&  \
    cp /usr/bin/envsubst /usr/local/bin/envsubst && \
    apk del build_deps

RUN envsubst ${GCP_PROJECT} ${GCP_BUCKET_NAME} ${GCP_BUCKET_KEY} ${CKAN_SITE_URL} < "/etc/giftless.yaml" > "/etc/giftless.yaml"

CMD ["-s", "127.0.0.1:${PORT}", "-M", "-T", "--threads", "2", "-p", "2", \
     "--manage-script-name", "--callable", "app"]