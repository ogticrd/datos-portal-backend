###################
### Extensions ####
###################
FROM keitaro/ckan:2.9.1 as extbuild

# Switch to the root user
USER root

# Install any system packages necessary to build extensions
RUN apk add --no-cache python3-dev

# Fetch and build the custom CKAN extensions
# RUN pip wheel --wheel-dir=/wheels git+https://github.com/acmecorp/ckanext-acme@0.0.1#egg=ckanext-acme

############
### MAIN ###
############
FROM keitaro/ckan:2.9.1 as release

# Add the custom extensions to the plugins list
ENV CKAN__PLUGINS envvars image_view text_view recline_view datastore datapusher

# Switch to the root user
USER root

COPY --from=extbuild /wheels /srv/app/ext_wheels

# Install and enable the custom extensions
RUN pip install --no-index --find-links=/srv/app/ext_wheels ckanext-acme && \
    ckan -c ${APP_DIR}/production.ini config-tool "ckan.plugins = ${CKAN__PLUGINS}" && \
    chown -R ckan:ckan /srv/app

# Remove wheels
RUN rm -rf /srv/app/ext_wheels

# Switch to the ckan user
USER ckan