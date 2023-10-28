# Alpine Linux
FROM alpine

# Latest version and commit hash, to be provided at build time with option `--build-arg`;
# `docker build -t olange/svn-server --build-arg GIT_SHA=$(git rev-parse --short HEAD) .`
ARG GIT_SHA
ARG SVN_VERSION

LABEL version=$SVN_VERSION
LABEL description="A simple Subversion server Docker image, based on Alpine Linux, with Apache HTTPD"
LABEL maintainer="Olivier Lange (https://github.com/olange)"
LABEL vcs-url="https://github.com/olange/containers/tree/main/svn-server"
LABEL vcs-ref=$GIT_SHA
LABEL docker.cmd="docker run -d -p 80:80 -p 443:443 -v ‹host_svnroot›:/home/svn --name svn-server olange/svn-server"

ENV SVN_ROOT_DIR /home/svn
ENV SVN_CONF_DIR /etc/subversion
ENV SVN_PASSWD_FILE $SVN_CONF_DIR/svn-auth-file.htpasswd
ENV SVN_AUTHZ_FILE $SVN_CONF_DIR/svn-authz-file.conf

ENV HTTPD_CONF_FILE /etc/apache2/httpd.conf
ENV HTTPD_CONF_DAV_SVN_FILE /etc/apache2/conf.d/dav_svn.conf
ENV HTTPD_LOGS_DIR /var/log/apache2
ENV HTTPD_PID_DIR /run/apache2/

# Set HOME in non /root folder
ENV HOME /home

# Create folders for Subversion config and repositories
RUN mkdir -p $HTTPD_PID_DIR &&\
    mkdir -p $SVN_CONF_DIR &&\
    mkdir -p $SVN_ROOT_DIR &&\
    chmod a+w $SVN_ROOT_DIR

# Install Apache2 and modules needed to access svn via WebDav
RUN apk add --no-cache apache2 apache2-utils \
                       apache2-webdav mod_dav_svn &&\
    apk add --no-cache subversion

# Solve a security issue (https://alpinelinux.org/posts/Docker-image-vulnerability-CVE-2019-5021.html)
RUN sed -i -e 's/^root::/root:!:/' /etc/shadow

# Apache default Mutex is not supported by Alpine Linux, switch to POSIX semaphores
RUN echo "Mutex posixsem" >> $HTTPD_CONF_FILE

# Redirect Apache logs to console (that is, Docker log collector)
RUN ln -sf /dev/stdout $HTTPD_LOGS_DIR/access.log &&\
    ln -sf /dev/stderr $HTTPD_LOGS_DIR/error.log

# Add the authentication SVNAuth & SVNAuthZ files for HTTP access
COPY --chmod=a+w etc/subversion/svn-auth-file.htpasswd $SVN_PASSWD_FILE
COPY --chmod=a+w etc/subversion/svn-authz-file.conf $SVN_AUTHZ_FILE

# Add the WebDAV configuration for Subversion
COPY etc/apache/dav_svn.conf $HTTPD_CONF_DAV_SVN_FILE

# Volumes for SVN repositories and configuration
VOLUME $SVN_ROOT_DIR

# https://httpd.apache.org/docs/2.4/stopping.html#gracefulstop
STOPSIGNAL SIGWINCH

EXPOSE 80 443

COPY --chmod=ug+x ./usr/local/bin/httpd-foreground /usr/local/bin/
CMD ["httpd-foreground"]
