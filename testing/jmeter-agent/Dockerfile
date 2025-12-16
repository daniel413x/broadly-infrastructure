FROM eclipse-temurin:11-jre

ENV JMETER_VERSION=5.6.3
ENV JMETER_HOME=/opt/apache-jmeter-$JMETER_VERSION
ENV PATH=$JMETER_HOME/bin:$PATH

RUN apt-get update && \
    apt-get install -y wget unzip iputils-ping bash && \
    apt-get clean

# install JMeter
RUN wget https://downloads.apache.org/jmeter/binaries/apache-jmeter-$JMETER_VERSION.tgz && \
    tar -xzf apache-jmeter-$JMETER_VERSION.tgz -C /opt && \
    rm apache-jmeter-$JMETER_VERSION.tgz


# --- install JMeter Plugins Manager + CmdRunner ---
# 
RUN wget -q https://repo1.maven.org/maven2/kg/apc/jmeter-plugins-manager/1.10/jmeter-plugins-manager-1.10.jar \
       -O $JMETER_HOME/lib/ext/jmeter-plugins-manager.jar && \
    wget -q https://repo1.maven.org/maven2/kg/apc/cmdrunner/2.3/cmdrunner-2.3.jar \
       -O $JMETER_HOME/lib/cmdrunner-2.3.jar && \
    java -cp $JMETER_HOME/lib/ext/jmeter-plugins-manager.jar \
       org.jmeterplugins.repository.PluginManagerCMDInstaller


# install required plugins
RUN java -jar $JMETER_HOME/lib/cmdrunner-2.3.jar \
      --tool org.jmeterplugins.repository.PluginManagerCMD install \
      jpgc-casutg,jpgc-tst,jpgc-dummy,jpgc-ffw,jpgc-json,jpgc-perfmon,jpgc-prmctl





WORKDIR $JMETER_HOME

# expose ports needed for distributed mode
EXPOSE 1099 50000


ENV SERVER_PORT=1099
ENV RMI_PORT=50000

ENV JVM_ARGS="-Dserver.rmi.ssl.disable=true -Dclient.rmi.ssl.disable=true -Dserver_port=1099 -Dserver.rmi.localport=50000"

# RMI settings for distributed JMeter
ENV JMETER_SERVER_JVM_ARGS="\
  -Dserver.rmi.ssl.disable=true \
  -Dclient.rmi.ssl.disable=true \
  -Dserver_port=1099 \
  -Dserver.rmi.localport=50000"




CMD ["jmeter-server"]
