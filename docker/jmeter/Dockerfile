FROM justb4/jmeter:5.6.3

USER root

ENV JMETER_PLUGINS_MANAGER_VERSION=1.10
ENV CMDRUNNER_VERSION=2.3

# Install Plugins Manager
RUN cd /opt/apache-jmeter-${JMETER_VERSION}/lib/ext && \
    curl -L -o jmeter-plugins-manager-${JMETER_PLUGINS_MANAGER_VERSION}.jar \
    https://jmeter-plugins.org/get/ && \
    curl -L -o /opt/apache-jmeter-${JMETER_VERSION}/lib/cmdrunner-${CMDRUNNER_VERSION}.jar \
    https://repo1.maven.org/maven2/kg/apc/cmdrunner/${CMDRUNNER_VERSION}/cmdrunner-${CMDRUNNER_VERSION}.jar && \
    java -cp jmeter-plugins-manager-${JMETER_PLUGINS_MANAGER_VERSION}.jar \
    org.jmeterplugins.repository.PluginManagerCMDInstaller

# Install plugins
COPY plugins.txt /tmp/plugins.txt
RUN /opt/apache-jmeter-${JMETER_VERSION}/bin/PluginsManagerCMD.sh install $(cat /tmp/plugins.txt | tr '\n' ',') && \
    rm /tmp/plugins.txt

USER jmeter

WORKDIR /tests