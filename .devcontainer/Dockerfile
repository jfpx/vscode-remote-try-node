#-------------------------------------------------------------------------------------------------------------
# Copyright (c) Microsoft Corporation. All rights reserved.
# Licensed under the MIT License. See https://go.microsoft.com/fwlink/?linkid=2090316 for license information.
#-------------------------------------------------------------------------------------------------------------

FROM node:11

# The node image includes a non-root user with sudo access. Use the "remoteUser"
# property in devcontainer.json to use it. On Linux, the container user's GID/UIDs
# will be updated to match your local UID/GID (when using the dockerFile property).
# See https://aka.ms/vscode-remote/containers/non-root-user for details.
ARG USERNAME=node
ARG USER_UID=1000
ARG USER_GID=$USER_UID

# Avoid warnings by switching to noninteractive
ENV DEBIAN_FRONTEND=noninteractive

# Configure apt and install packages
RUN apt-get update \
    && apt-get -y install --no-install-recommends apt-utils dialog 2>&1 \ 
    #
    # Verify git and needed tools are installed
    && apt-get -y install git iproute2 procps \
    #
    # Remove outdated yarn from /opt and install via package 
    # so it can be easily updated via apt-get upgrade yarn
    && rm -rf /opt/yarn-* \
    && rm -f /usr/local/bin/yarn \
    && rm -f /usr/local/bin/yarnpkg \
    && apt-get install -y curl apt-transport-https lsb-release \
    && curl -sS https://dl.yarnpkg.com/$(lsb_release -is | tr '[:upper:]' '[:lower:]')/pubkey.gpg | apt-key add - 2>/dev/null \
    && echo "deb https://dl.yarnpkg.com/$(lsb_release -is | tr '[:upper:]' '[:lower:]')/ stable main" | tee /etc/apt/sources.list.d/yarn.list \
    && apt-get update \
    && apt-get -y install --no-install-recommends yarn \
    #
    # Install eslint globally
    && npm install -g eslint \
    #
    # [Optional] Update a non-root user to UID/GID if needed.
    && if [ "$USER_GID" != "1000" ] || [ "$USER_UID" != "1000" ]; then \
        groupmod --gid $USER_GID $USERNAME \
        && usermod --uid $USER_UID --gid $USER_GID $USERNAME \
        && chown -R $USER_UID:$USER_GID /home/$USERNAME; \
    fi \
    # [Optional] Add add sudo support for non-root user
    && apt-get install -y sudo \
    && echo node ALL=\(root\) NOPASSWD:ALL > /etc/sudoers.d/$USERNAME \
    && chmod 0440 /etc/sudoers.d/$USERNAME \
    #
    # Clean up
    && apt-get autoremove -y \
    && apt-get clean -y \
    && rm -rf /var/lib/apt/lists/*


RUN wget https://yt-dl.org/downloads/latest/youtube-dl -O /usr/bin/youtube-dl \
    && chmod a+rx /usr/bin/youtube-dl

RUN apt-get update \
    && apt-get install -yq --no-install-recommends \
        git wget curl gnupg2 libgconf-2-4 python3 python3-pip \
    && rm -rf /var/lib/apt/lists/*


# Install latest chrome package and fonts to support major charsets (Chinese, Japanese, Arabic, Hebrew, Thai and a few others)
RUN apt-get update && apt-get install -y wget --no-install-recommends \
    && wget -q -O - https://dl-ssl.google.com/linux/linux_signing_key.pub | apt-key add - \
    && sh -c 'echo "deb [arch=amd64] http://dl.google.com/linux/chrome/deb/ stable main" >> /etc/apt/sources.list.d/google.list' \
    && apt-get update \
    && apt-get install -y google-chrome-unstable fonts-ipafont-gothic fonts-wqy-zenhei fonts-thai-tlwg fonts-kacst ttf-freefont \
      --no-install-recommends \
    && rm -rf /var/lib/apt/lists/* \
    && rm -rf /src/*.deb

# It's a good idea to use dumb-init to help prevent zombie chrome processes.
ADD https://github.com/Yelp/dumb-init/releases/download/v1.2.0/dumb-init_1.2.0_amd64 /usr/local/bin/dumb-init
RUN chmod +x /usr/local/bin/dumb-init

# Uncomment to skip the chromium download when installing puppeteer. If you do,
# you'll need to launch puppeteer with:
#     browser.launch({executablePath: 'google-chrome-unstable'})
ENV PUPPETEER_SKIP_CHROMIUM_DOWNLOAD true

# Install puppeteer so it's available in the container.
RUN npm i puppeteer

# Add user so we don't need --no-sandbox.  groupadd -r $USERNAME && useradd -r -g $USER_UID -G audio,video $USERNAME \
RUN  mkdir -p /home/$USERNAME/Downloads \
    && chown -R $USER_UID:$USER_GID /home/$USERNAME \
    && chown -R $USER_UID:$USER_GID /node_modules

# Install the ArchiveBox repository and pip requirements
COPY . /home/$USERNAME/app
RUN mkdir -p /data \
    && chown -R $USER_UID:$USER_GID /data \
    && mkdir -p /home/$USERNAME/app/archivebox/output \
    && chown -R $USER_UID:$USER_GID /home/$USERNAME/app/archivebox/output \
    && ln -s /data /home/$USERNAME/app/archivebox/output
    
RUN ln -s /home/$USERNAME/app/bin/* /bin/
RUN ln -s /home/$USERNAME/app/bin/archivebox /bin/archive
RUN chown -R $USER_UID:$USER_GID /home/$USERNAME/app/archivebox

VOLUME /data

ENV LANG=C.UTF-8 \
    LANGUAGE=en_US:en \
    LC_ALL=C.UTF-8 \
    PYTHONIOENCODING=UTF-8 \
    CHROME_SANDBOX=False \
    CHROME_BINARY=google-chrome-unstable \
    OUTPUT_DIR=/data \
    MEDIA_TIMEOUT=36000 \
    TIMEOUT=36000 \
    FETCH_MEDIA=True \ 
    CHECK_SSL_VALIDITY=False

# Run everything from here on out as non-privileged user
USER $USERNAME
WORKDIR /home/$USERNAME/app

# Switch back to dialog for any ad-hoc use of apt-get
ENV DEBIAN_FRONTEND=dialog

ENTRYPOINT ["dumb-init", "--"]
CMD ["/bin/archive"]
