FROM ubuntu:22.04

# PostgreSQL Version Configuration - Change this to upgrade PostgreSQL
ARG POSTGRES_VERSION=15
ENV POSTGRES_VERSION=${POSTGRES_VERSION}

# Avoid warnings by switching to noninteractive
ENV DEBIAN_FRONTEND=noninteractive

# Add PostgreSQL official APT repository for specific version control
RUN apt-get update \
    && apt-get -y install --no-install-recommends apt-utils dialog wget ca-certificates gnupg lsb-release 2>&1 \
    && wget --quiet -O - https://www.postgresql.org/media/keys/ACCC4CF8.asc | apt-key add - \
    && echo "deb http://apt.postgresql.org/pub/repos/apt/ $(lsb_release -cs)-pgdg main" > /etc/apt/sources.list.d/pgdg.list \
    && apt-get update

# Configure apt and install packages with specific PostgreSQL version
RUN apt-get -y install --no-install-recommends \
    git \
    curl \
    wget \
    software-properties-common \
    build-essential \
    libssl-dev \
    libreadline-dev \
    zlib1g-dev \
    libsqlite3-dev \
    sqlite3 \
    postgresql-${POSTGRES_VERSION} \
    postgresql-contrib-${POSTGRES_VERSION} \
    postgresql-client-${POSTGRES_VERSION} \
    libpq-dev \
    python3 \
    python3-pip \
    vim \
    nano \
    sudo \
    #
    # Clean up
    && apt-get autoremove -y \
    && apt-get clean -y \
    && rm -rf /var/lib/apt/lists/*

# Install Node.js 18 (LTS) for VueJS
RUN curl -fsSL https://deb.nodesource.com/setup_18.x | bash - \
    && apt-get install -y nodejs

# Configure npm to use user directory for global packages
RUN npm config set prefix ~/.npm-global \
    && echo 'export PATH=~/.npm-global/bin:$PATH' >> ~/.bashrc

# Create a non-root user
ARG USERNAME=vscode
ARG USER_UID=1000
ARG USER_GID=$USER_UID

RUN groupadd --gid $USER_GID $USERNAME \
    && useradd --uid $USER_UID --gid $USER_GID -m $USERNAME \
    && echo $USERNAME ALL=\(root\) NOPASSWD:ALL > /etc/sudoers.d/$USERNAME \
    && chmod 0440 /etc/sudoers.d/$USERNAME

# Configure PostgreSQL with dynamic version paths
RUN service postgresql start \
    && sudo -u postgres createuser -s vscode \
    && sudo -u postgres createdb vscode \
    && sudo -u postgres psql -c "ALTER USER vscode PASSWORD 'password';" \
    && echo "local all vscode trust" >> /etc/postgresql/${POSTGRES_VERSION}/main/pg_hba.conf \
    && echo "host all vscode 127.0.0.1/32 trust" >> /etc/postgresql/${POSTGRES_VERSION}/main/pg_hba.conf \
    && echo "host all vscode ::1/128 trust" >> /etc/postgresql/${POSTGRES_VERSION}/main/pg_hba.conf \
    && service postgresql stop

# Switch to non-root user
USER $USERNAME
WORKDIR /home/$USERNAME

# Create startup script with PostgreSQL version awareness
RUN echo '#!/bin/bash' > /home/vscode/start-services.sh \
    && echo "# PostgreSQL Version: ${POSTGRES_VERSION}" >> /home/vscode/start-services.sh \
    && echo 'sudo service postgresql restart' >> /home/vscode/start-services.sh \
    && echo 'sleep 2' >> /home/vscode/start-services.sh \
    && echo '# Verify PostgreSQL is running' >> /home/vscode/start-services.sh \
    && echo 'sudo service postgresql status >/dev/null 2>&1 || { echo "PostgreSQL failed to start"; exit 1; }' >> /home/vscode/start-services.sh \
    && echo 'exec "$@"' >> /home/vscode/start-services.sh \
    && chmod +x /home/vscode/start-services.sh

# Set up the working directory
WORKDIR /workspace

# Switch back to dialog for any ad-hoc use of apt-get
ENV DEBIAN_FRONTEND=dialog

# Set database environment variables for Rails with version info
ENV DATABASE_URL="postgresql://vscode:password@localhost/vscode"
ENV POSTGRES_USER="vscode"
ENV POSTGRES_PASSWORD="password"
ENV POSTGRES_DB="vscode"
ENV POSTGRES_VERSION_INFO="PostgreSQL ${POSTGRES_VERSION}"

# Start PostgreSQL by default
ENTRYPOINT ["/home/vscode/start-services.sh"]
CMD ["/bin/bash"] 