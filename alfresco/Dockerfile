ARG ALFRESCO_TAG
FROM alfresco/alfresco-content-repository-community:${ALFRESCO_TAG}

ARG TOMCAT_DIR=/usr/local/tomcat
ARG IMAGEUSERNAME=alfresco

# default user is alfresco (added on the base image alfresco/alfresco-content-repository-community)
# change to root user to be able to install the addons and packages
USER root

# Install modules and addons
RUN mkdir -p $TOMCAT_DIR/amps
COPY modules/amps $TOMCAT_DIR/amps
COPY modules/jars $TOMCAT_DIR/webapps/alfresco/WEB-INF/lib

RUN java -jar $TOMCAT_DIR/alfresco-mmt/alfresco-mmt*.jar install \
    $TOMCAT_DIR/amps $TOMCAT_DIR/webapps/alfresco -directory -nobackup -force



# DATABASE
ARG DB
ENV DB $DB

# Install mysql JDBC driver
RUN if [ "$DB" == "mariadb" ] ; then \
    set -x \
        && yum install -y wget \
        && yum clean all \
        && wget -P /tmp/ https://repo1.maven.org/maven2/org/mariadb/jdbc/mariadb-java-client/2.7.4/mariadb-java-client-2.7.4.jar \
        && cp /tmp/mariadb-java-client-2.7.4.jar $TOMCAT_DIR/lib/ \
        && rm -rf /tmp/mariadb-java-client-2.7.4.jar; \
fi


# SSH keys for ocrmypdf
COPY ssh/ /root/.ssh/

# Install OCR
COPY bin/ /opt/alfresco/bin/

# Configure SSH Client
RUN set -x && \
    chmod +x /opt/alfresco/bin/ocrmypdf.sh && \
    # Configure ssh
    yum install -y openssh-clients && \
    echo "StrictHostKeyChecking no" >> /etc/ssh/ssh_config && \
    # Alfresco Image is using POSIX as Locale (!)
    sed -i '/^\s*SendEnv/ d' /etc/ssh/ssh_config && \
    chmod 600 /root/.ssh/id_rsa



EXPOSE 2121 2433 2434


# Restore original user
RUN chown -R ${IMAGEUSERNAME} $TOMCAT_DIR
USER ${IMAGEUSERNAME}
