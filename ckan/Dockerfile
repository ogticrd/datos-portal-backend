###################
### Extensions ####
###################
FROM ghcr.io/keitaroinc/ckan:2.9.4 as extbuild

# Locations and tags, please use specific tags or revisions
ENV HARVEST_GIT_URL=https://github.com/ckan/ckanext-harvest
ENV HARVEST_GIT_BRANCH=v1.3.1

ENV BLOB_GIT_URL=https://github.com/datopian/ckanext-blob-storage
ENV BLOB_GIT_BRANCH=v0.8.0

ENV AUTHZ_GIT_URL=https://github.com/datopian/ckanext-authz-service
ENV AUTHZ_GIT_BRANCH=v0.1.5

ENV ASSET_GIT_URL=https://github.com/datopian/ckanext-asset-storage
ENV ASSET_GIT_BRANCH=v0.1.6

# Switch to the root user
USER root

# Make sure we install python 3.8, cause CKAN is not compatible with 3.9
RUN apk add --no-cache --repository=http://dl-cdn.alpinelinux.org/alpine/v3.13/main \
        python3-dev=3.8.10-r0 && \
    apk add --update --no-cache \
        gcc \
        g++ \
        libffi-dev \
        openssl-dev \
        rust \
        cargo \
        nodejs \
        npm

# Fetch and build the custom CKAN extensions
RUN pip wheel --wheel-dir=/wheels git+${HARVEST_GIT_URL}@${HARVEST_GIT_BRANCH}#egg=ckanext-harvest
RUN pip wheel --wheel-dir=/wheels -r https://raw.githubusercontent.com/ckan/ckanext-harvest/${HARVEST_GIT_BRANCH}/pip-requirements.txt
RUN curl -o /wheels/harvest.txt https://raw.githubusercontent.com/ckan/ckanext-harvest/${HARVEST_GIT_BRANCH}/pip-requirements.txt

RUN pip wheel --wheel-dir=/wheels git+${BLOB_GIT_URL}@${BLOB_GIT_BRANCH}#egg=ckanext-blob-storage
RUN pip wheel --wheel-dir=/wheels -r https://raw.githubusercontent.com/datopian/ckanext-blob-storage/${BLOB_GIT_BRANCH}/requirements.py3.txt
RUN curl -o /wheels/blob_storage.txt https://raw.githubusercontent.com/ckan/ckanext-harvest/${BLOB_GIT_BRANCH}/pip-requirements.txt

RUN pip wheel --wheel-dir=/wheels git+${AUTHZ_GIT_URL}@${AUTHZ_GIT_BRANCH}#egg=ckanext-authz-service
RUN pip wheel --wheel-dir=/wheels -r https://raw.githubusercontent.com/datopian/ckanext-authz-service/${AUTHZ_GIT_BRANCH}/requirements.py3.txt
RUN curl -o /wheels/authz_service.txt https://raw.githubusercontent.com/datopian/ckanext-authz-service/${AUTHZ_GIT_BRANCH}/requirements.py3.txt

RUN pip wheel --wheel-dir=/wheels git+${ASSET_GIT_URL}@${ASSET_GIT_BRANCH}#egg=ckanext-asset-storage
RUN pip wheel --wheel-dir=/wheels -r https://raw.githubusercontent.com/datopian/ckanext-asset-storage/${ASSET_GIT_BRANCH}/requirements.py3.txt
RUN curl -o /wheels/asset_storage.txt https://raw.githubusercontent.com/datopian/ckanext-asset-storage/${ASSET_GIT_BRANCH}/requirements.py3.txt

USER ckan

############
### MAIN ###
############
FROM ghcr.io/keitaroinc/ckan:2.9.4 as release

LABEL maintainer="Keitaro Inc <info@keitaro.com>"

# Add the custom extensions to the plugins list
ENV CKAN__PLUGINS envvars image_view text_view recline_view datastore datapusher \
    harvest ckan_harvester blob_storage authz_service asset_storage

# Switch to the root user
USER root

COPY --from=extbuild /wheels /srv/app/ext_wheels

# Install and enable the custom extensions
RUN pip install --no-index --find-links=/srv/app/ext_wheels ckanext-harvest && \
    pip install --no-index --find-links=/srv/app/ext_wheels -r /srv/app/ext_wheels/harvest.txt && \
    pip install --no-index --find-links=/srv/app/ext_wheels ckanext-blob-storage && \
    pip install --no-index --find-links=/srv/app/ext_wheels -r /srv/app/ext_wheels/blob_storage.txt && \
    pip install --no-index --find-links=/srv/app/ext_wheels ckanext-authz-service && \
    pip install --no-index --find-links=/srv/app/ext_wheels -r /srv/app/ext_wheels/authz_service.txt && \
    pip install --no-index --find-links=/srv/app/ext_wheels ckanext-asset-storage && \
    pip install --no-index --find-links=/srv/app/ext_wheels -r /srv/app/ext_wheels/asset_storage.txt && \
    # Configure plugins
    ckan config-tool "${APP_DIR}/production.ini" "ckan.plugins = ${CKAN__PLUGINS}" && \
    ckan config-tool "${APP_DIR}/production.ini" "ckanext.asset_storage.backend_type = google_cloud" && \
    chown -R ckan:ckan /srv/app

# Remove wheels
RUN rm -rf /srv/app/ext_wheels

# Add harvest afterinit script
COPY ./afterinit.d/00_harvest.sh ${APP_DIR}/docker-afterinit.d/00_harvest.sh

# Switch to the ckan user
USER ckan