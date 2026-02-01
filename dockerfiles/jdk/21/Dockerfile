# Start from the official Jenkins image
FROM localhost/jenkins-node:nodejs-24

# Switch to root to install packages
USER root

# Install Node.js (example for Node.js 24 LTS)
RUN apt-get install -y --no-install-recommends openjdk-21-jdk \
    && apt-get clean \
    && rm -rf /var/lib/apt/lists/*

ENV JAVA_HOME=/usr/lib/jvm/java-21-openjdk-amd64
ENV PATH="$JAVA_HOME/bin:$PATH"

CMD ["java","-version"]

# Switch back to Jenkins user for security
USER jenkins
