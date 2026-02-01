# Start from the official Jenkins image
FROM jenkins/jenkins:lts-jdk21

# Switch to root to install packages
USER root

# Install Node.js (example for Node.js 24 LTS)
RUN curl -fsSL https://deb.nodesource.com/setup_24.x | bash - \
    && apt-get install -y nodejs \
    && apt-get clean \
    && rm -rf /var/lib/apt/lists/*

# Optionally install npm packages globally
RUN npm install -g npm
RUN npm install -g gulp
RUN npm install -g rimraf

# Switch back to Jenkins user for security
USER jenkins

# Set default workdir for Jenkins (optional)
WORKDIR /var/jenkins_home
