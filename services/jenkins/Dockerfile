FROM jenkins/jenkins:2.516.3-jdk17
USER root

RUN curl -fsSL https://bun.sh/install | bash -s -- \
# because jenkins-pvc is mounted to /var/jenkins_home, installations of bun will be overwritten on deployment and thus must be moved
# ensure bun cmd works
&& install -m 0755 /root/.bun/bin/bun  /usr/local/bin/bun \
# ensure bunx cmd works
&& install -m 0755 /root/.bun/bin/bunx /usr/local/bin/bunx \
&& rm -rf /root/.bun

RUN apt-get update && apt-get install -y lsb-release maven zip
RUN curl -fsSLo /usr/share/keyrings/docker-archive-keyring.asc \
  https://download.docker.com/linux/debian/gpg
RUN echo "deb [arch=$(dpkg --print-architecture) \
  signed-by=/usr/share/keyrings/docker-archive-keyring.asc] \
  https://download.docker.com/linux/debian \
  $(lsb_release -cs) stable" > /etc/apt/sources.list.d/docker.list
RUN curl -fsSL https://deb.nodesource.com/setup_16.x | bash -
# kubernetes-client = kubectl
RUN apt-get update && apt-get install -y docker-ce-cli nodejs kubernetes-client
RUN export M2_HOME=/usr/bin/mvn; export MAVEN_HOME=/usr/bin/mvn
RUN apt-get update && apt-get install -y chromium chromium-driver
RUN curl "https://awscli.amazonaws.com/awscli-exe-linux-x86_64.zip" -o "awscliv2.zip"
RUN unzip awscliv2.zip
RUN apt install docker.io -y
RUN ./aws/install

# performance testing -- install bzt (Taurus) dependencies 
RUN apt-get update \
&& apt-get install -y python3 default-jre-headless python3-tk python3-pip python3-dev libxml2-dev libxslt-dev zlib1g-dev net-tools python3-virtualenv

# install Taurus
RUN pip3 install --no-cache-dir --break-system-packages bzt

RUN jenkins-plugin-cli --plugins "maven-plugin workflow-aggregator git sonar nodejs kubernetes aws-credentials docker-workflow ssh-agent email-ext blueocean pipeline-aws docker-plugin configuration-as-code job-dsl workflow-multibranch scm-api branch-api http_request"
