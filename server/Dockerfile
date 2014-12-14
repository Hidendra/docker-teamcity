FROM dockerfile/java:oracle-java7
MAINTAINER clayman <clayman74@gmail.com>

ENV TEAMCITY_PACKAGE TeamCity-9.0.tar.gz
ENV TEAMCITY_DOWNLOAD http://download.jetbrains.com/teamcity

RUN apt-get -y install tar wget

RUN cd /opt && wget $TEAMCITY_DOWNLOAD/$TEAMCITY_PACKAGE && tar xvzf $TEAMCITY_PACKAGE && rm -f $TEAMCITY_PACKAGE

VOLUME  ["/root"]

EXPOSE 8111

CMD ["/opt/TeamCity/bin/teamcity-server.sh", "run"]