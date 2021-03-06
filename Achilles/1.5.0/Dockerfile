# A Docker container to run the OHDSI/Achilles analysis tool
FROM ubuntu:trusty

ARG ACHILLES_VERSION=1.5.0

MAINTAINER Aaron Browne <brownea@email.chop.edu>

# Install java, R and required packages and clean up.
RUN echo deb http://ppa.launchpad.net/marutter/rrutter/ubuntu trusty main >> /etc/apt/sources.list && \
    echo deb http://ppa.launchpad.net/marutter/c2d4u/ubuntu trusty main >> /etc/apt/sources.list && \
    apt-key adv --keyserver keyserver.ubuntu.com --recv-keys C9A7585B49D51698710F3A115E25F516B04C661B && \
    sed 's#http://.*archive\.ubuntu\.com/ubuntu/#mirror://mirrors.ubuntu.com/mirrors.txt#g' -i /etc/apt/sources.list && \
    apt-get update && \
    DEBIAN_FRONTEND=noninteractive apt-get install -y \
      r-base \
      r-cran-devtools \
      r-cran-httr \
      r-cran-rjson \
      r-cran-stringr \
      r-cran-rjava \
      r-cran-dbi \
      r-cran-ffbase \
      littler \
      openjdk-7-jdk \
    && rm -rf /var/lib/apt/lists/* \
    && R CMD javareconf

# Set default locale
RUN echo "en_US.UTF-8 UTF-8" >> /etc/locale.gen \
	&& locale-gen en_US.utf8 \
	&& /usr/sbin/update-locale LANG=en_US.UTF-8

ENV LC_ALL en_US.UTF-8
ENV LANG en_US.UTF-8

# Install Achilles requirements that need to be installed from source
RUN echo 'options(repos=structure(c(CRAN="http://cran.cnr.berkeley.edu/")))' > /root/.Rprofile && \
    /usr/share/doc/littler/examples/install.r remotes && \
    /usr/share/doc/littler/examples/install.r docopt && \
    /usr/share/doc/littler/examples/installGithub.r \
      OHDSI/SqlRender \
      OHDSI/DatabaseConnector \
    && rm -rf /tmp/downloaded_packages/ /tmp/*.rds

# Configure workspace
WORKDIR /opt/app
ENV PATH /opt/app:$PATH
VOLUME /opt/app/output

# Add project files to container
RUN curl -fsSL -o /tmp/achilles.tar.gz https://github.com/trberg/Achilles/archive/v${ACHILLES_VERSION}.tar.gz
RUN tar -xzf /tmp/achilles.tar.gz -C /opt/app --strip-components=1

# Install Achilles from source
RUN R CMD INSTALL /opt/app \
    && rm -rf /tmp/downloaded_packages/ /tmp/*.rds \
    && find /opt/app -mindepth 1 -not \( -wholename /opt/app/docker-run -or -wholename /opt/app/output \) -delete

# Define run script as default command
CMD ["docker-run"]
