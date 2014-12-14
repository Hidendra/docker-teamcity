FROM dockerfile/java:oracle-java7
MAINTAINER clayman <clayman74@gmail.com>

# Add UTF-8 locale
RUN locale-gen en_US.UTF-8 && update-locale LANG=en_US.UTF-8

# Update system and install required libraries
RUN apt-get update -y && apt-get -y dist-upgrade && apt-get -y upgrade
RUN apt-get install -y wget git vim python-dev python-virtualenv python-pip libjpeg62-dev zlib1g-dev libpq-dev libxslt1-dev libxml2-dev
RUN pip install --upgrade pip

# prepate to setup build agent
ADD setup-agent.sh /setup-agent.sh
RUN apt-get install -y wget unzip sudo

VOLUME ["/root"]

EXPOSE 9090

CMD /bin/bash /setup-agent.sh
