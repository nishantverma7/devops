FROM ubuntu:16.04
MAINTAINER HotWax Commerce (sandeep.kose@hotwaxsystems.com)

#Install Open JDK 8
RUN apt-get update && apt-get -y install openjdk-8-jdk ca-certificates-java git wget  gnupg bash vim screen

ENV JAVA_HOME /usr/lib/jvm/java-8-openjdk-amd64
ENV PATH $JAVA_HOME/bin:$PATH

ARG GIT_USERNAME=
ARG GIT_PASSWORD=
ARG Moqui_HOST=
ARG Moqui_DB_HOST=
ARG Moqui_DB_USER=
ARG Moqui_DB_PASSWORD=
ARG Moqui_DB_NAME=
ARG Moqui_analytical_DB_Name=
ARG Moqui_configuration_DB_Name=
ARG ELASTICSEARCH_HOST=

RUN echo "machine git.hotwax.co" > /root/.netrc
RUN echo "login $GIT_USERNAME" >> /root/.netrc
RUN echo "password $GIT_PASSWORD" >> /root/.netrc

WORKDIR /
RUN wget https://archive.apache.org/dist/tomcat/tomcat-8/v8.5.51/bin/apache-tomcat-8.5.51.tar.gz
RUN tar -xzf apache-tomcat-8.5.51.tar.gz
RUN mv apache-tomcat-8.5.51 apache-tomcat
RUN rm -rf apache-tomcat/webapps/ROOT
COPY server.xml /root
COPY hotwaxco.jks /root/
RUN cp -rf /root/server.xml /apache-tomcat/conf/server.xml

RUN git clone -b "v3.1.0" https://git.hotwax.co/HC2/commerce/moqui-framework.git

WORKDIR /moqui-framework
RUN git clone -b "v3.1.0" https://git.hotwax.co/HC2/commerce/moqui-runtime.git runtime

WORKDIR /moqui-framework/runtime/component
RUN git clone -b "v3.1.0" https://git.hotwax.co/HC2/commerce/mantle-udm.git
RUN git clone -b "v3.1.1" https://git.hotwax.co/HC2/commerce/mantle-usl.git
RUN git clone -b "v3.1.0" https://git.hotwax.co/HC2/commerce/SimpleScreens.git
RUN git clone -b "v1.1.3" https://git.hotwax.co/HC2/plugins/moqui-fop.git
RUN git clone -b "v1.0.3" https://git.hotwax.co/HC2/plugins/moqui-kie.git
RUN git clone -b "v1.1.1" https://git.hotwax.co/HC2/plugins/moqui-sftp.git
RUN git clone -b "v1.9.1" https://git.hotwax.co/HC2/plugins/commerce-api.git
RUN git clone -b "v1.1.0" https://git.hotwax.co/HC2/plugins/orders.git
RUN git clone -b "v1.0.0" https://git.hotwax.co/HC2/plugins/shopify-connector.git
RUN git clone -b "v1.0.0" https://github.com/hotwax/mantle-stripe.git

RUN git clone -b "v1.0.1" https://git.hotwax.co/urbanrooster/urbanroosters.git

RUN rm -rf /root/.netrc
WORKDIR /moqui-framework
#RUN for i in runtime/component/*/patches/*.patch; do echo =========$i======; patch -p0 < $i; done
#Make it configurable using environment thing for dev/prod environment
RUN for i in runtime/component/*/patches/production/*.patch; do echo =========$i======; patch -p0 < $i; done

#Database Settings
RUN sed -i  's/127.0.0.1:9200/'$ELASTICSEARCH_HOST':9200/g' runtime/conf/MoquiProductionConf.xml
RUN sed -i -e 's/name="entity_ds_host" value="127.0.0.1"/name="entity_ds_host" value="'$Moqui_DB_HOST'"/g' runtime/conf/MoquiProductionConf.xml
RUN sed -i -e 's/name="webapp_http_host" value=""/name="webapp_http_host" value="'$Moqui_HOST'"/g' runtime/conf/MoquiProductionConf.xml

RUN sed -i 's/name="entity_ds_user" value="moqui"/name="entity_ds_user" value="'$Moqui_DB_USER'"/g' runtime/conf/MoquiProductionConf.xml
RUN sed -i 's/name="entity_ds_password" value="moqui"/name="entity_ds_password" value="'$Moqui_DB_PASSWORD'"/g' runtime/conf/MoquiProductionConf.xml
RUN sed -i 's/name="entity_ds_database" value="moqui"/name="entity_ds_database" value="'$Moqui_DB_NAME'"/g' runtime/conf/MoquiProductionConf.xml
RUN sed -i 's/name="entity_ds_database_analytical" value="moqui_analytical"/name="entity_ds_database_analytical" value="'$Moqui_analytical_DB_Name'"/g' runtime/conf/MoquiProductionConf.xml
RUN sed -i 's/name="entity_ds_database_configuration" value="moqui_configuration"/name="entity_ds_database_configuration" value="'$Moqui_configuration_DB_Name'"/g' runtime/conf/MoquiProductionConf.xml

ENV DB_LOAD="./gradlew loadProduction -Ptypes=seed,seed-initial,install,ext-seed,ext,ext-prod -Praw"
ENV JAVA_OPTS="-Xms512m -Xmx2048m"
ENV SLEEP="sleep 5"

EXPOSE 8443
ENTRYPOINT  $SLEEP && ./gradlew addRuntimeTomcat && cd /apache-tomcat/bin/ && export JAVA_OPTS="$JAVA_OPTS" CATALINA_OPTS="$JAVA_OPTS" && ./startup.sh && sleep 3  && tail -f /apache-tomcat/logs/catalina.out && bash
