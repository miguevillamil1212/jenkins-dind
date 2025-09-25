FROM jenkins/jenkins:lts-jdk17

# Instalar Docker CLI dentro de Jenkins
USER root
RUN apt-get update && apt-get install -y \
    docker.io \
    curl \
    && rm -rf /var/lib/apt/lists/*

# Volver al usuario jenkins
USER jenkins
