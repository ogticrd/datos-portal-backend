FROM datopian/giftless

ENV GIFTLESS_CONFIG_FILE=/etc/giftless.yaml
ENV GOOGLE_CLOUD_CREDENTIALS=/etc/google_credentials.json

# Set variables from build arguments
ARG PORT=5000
EXPOSE $PORT

ARG GCP_PROJECT
ENV GCP_PROJECT ${GCP_PROJECT}

ARG GCP_BUCKET_NAME
ENV GCP_BUCKET_NAME ${GCP_BUCKET_NAME}

ARG CKAN_SITE_URL
ENV CKAN_SITE_URL ${CKAN_SITE_URL}

# Use transformed enviroment variables from Giftless
ENV GIFTLESS_CONFIG_TRANSFER_ADAPTERS_BASIC_OPTIONS_STORAGE_OPTIONS_PROJECT_NAME ${GCP_PROJECT}
ENV GIFTLESS_CONFIG_TRANSFER_ADAPTERS_BASIC_OPTIONS_STORAGE_OPTIONS_BUCKET_NAME ${GCP_BUCKET_NAME}
ENV GIFTLESS_CONFIG_TRANSFER_ADAPTERS_BASIC_OPTIONS_STORAGE_OPTIONS_ACCOUNT_KEY ${GOOGLE_CLOUD_CREDENTIALS}

COPY giftless.yaml ${GIFTLESS_CONFIG_FILE}
COPY google_credentials.json ${GOOGLE_CLOUD_CREDENTIALS}

USER root

RUN --mount=type=secret,id=credentials,required,target=/etc/google_credentials.json \
  cat /etc/google_credentials.json

RUN DEBIAN_FRONTEND=noninteractive apt-get update \
    && apt-get install -y gettext-base \
    && apt-get clean \
    && apt -y autoremove

RUN envsubst "${GCP_PROJECT} ${GCP_BUCKET_NAME} ${GOOGLE_CLOUD_CREDENTIALS} ${CKAN_SITE_URL}" < ${GIFTLESS_CONFIG_FILE}

USER giftless

CMD ["-s", "127.0.0.1:${PORT}", "-M", "-T", "--threads", "2", "-p", "2", \
     "--manage-script-name", "--callable", "app"]