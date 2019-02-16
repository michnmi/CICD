FROM jenkins/ssh-slave
RUN apt-get install -y curl && \
    curl -sL https://deb.nodesource.com/setup_10.x | bash && \
    apt-get update -y && \
    apt-get install -y nodejs
