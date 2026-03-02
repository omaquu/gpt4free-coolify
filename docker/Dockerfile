# 1. Use a stable Selenium image with Python 3.12 (Avoids 3.14 Pydantic crash)
FROM selenium/node-chrome:4.25.0-20240922

ARG G4F_VERSION
ENV G4F_VERSION=$G4F_VERSION
ENV SE_SCREEN_WIDTH=1850
ENV G4F_DIR=/app
ENV VIRTUAL_ENV=/opt/venv         
ENV G4F_LOGIN_URL=http://localhost:7900/?autoconnect=1&resize=scale&password=secret
# Ensure the virtual env is always used for python/pip commands
ENV PATH="$VIRTUAL_ENV/bin:$HOME/.local/bin:$PATH"

USER root

# Install System Dependencies
RUN apt-get -qqy update && \
    apt-get -qqy upgrade && \
    apt-get -qqy install git python3 python3-pip python3-venv build-essential g++ python3-dev && \
    apt-get -qyy remove openjdk-11-jre-headless && \
    apt-get -qyy autoremove && \
    apt-get -qyy clean && \
    rm -rf /var/lib/apt/lists/* /var/cache/apt/*

# 2. Setup Virtual Environment correctly
RUN python3 -m venv $VIRTUAL_ENV && \
    chown -R $SEL_UID:$SEL_GID $VIRTUAL_ENV

# 3. Prepare User Workspace
RUN chown -R "${SEL_UID}:${SEL_GID}" $HOME/.local && \
    mkdir -p /home/seluser/.cache/pip && \
    chown -R $SEL_UID:$SEL_GID /home/seluser/.cache/pip

# Switch to non-root user for security and file ownership
USER $SEL_UID
WORKDIR $G4F_DIR

# 4. Install Python Dependencies
# We use $VIRTUAL_ENV/bin/pip to ensure packages go to the right place
COPY requirements.txt $G4F_DIR/
RUN $VIRTUAL_ENV/bin/pip install --upgrade pip && \
    $VIRTUAL_ENV/bin/pip install --no-cache-dir -r requirements.txt && \
    $VIRTUAL_ENV/bin/pip install --no-cache-dir a2wsgi g4f[api] spacy

# 5. Install spaCy Model (Now compatible with Python 3.12)
RUN $VIRTUAL_ENV/bin/python -m spacy download en_core_web_sm

# 6. Copy Application Files
ADD --chown=$SEL_UID:$SEL_GID g4f $G4F_DIR/g4f

# Setup internal directories for g4f
RUN mkdir -p $G4F_DIR/har_and_cookies $G4F_DIR/generated_images && \
    touch $G4F_DIR/har_and_cookies/.scrape_cache && \
    chmod -R 755 $G4F_DIR/har_and_cookies $G4F_DIR/generated_images && \
    chown -R $SEL_UID:$SEL_GID $G4F_DIR/har_and_cookies $G4F_DIR/generated_images

# 7. Configure Supervisor & UI
USER root
COPY docker/supervisor.conf /etc/supervisor/conf.d/selenium.conf
COPY docker/supervisor-api.conf /etc/supervisor/conf.d/api.conf
COPY docker/background.png /usr/share/images/fluxbox/ubuntu-light.png
USER $SEL_UID

# Expose ports: 10555 (API), 7900 (VNC/GUI), 1337 (G4F UI)
EXPOSE 10555 7900 1337
