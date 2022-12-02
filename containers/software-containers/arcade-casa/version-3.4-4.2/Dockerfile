FROM centos:5

# xterm dependency is an extra to get the casa shell in the display
# perl was added for casa later than 5

# Override old repo info with current urls
# NB: this revised process is completely eliminating two of the original repos
# which are CentOS-Media.repo, for configuring CD installation, and 
# CentOS-Vault.repo, which provides urls for all previous versions of 
# CentOS 5 (current versiou here is 5.10)
# Default baseurl also no longer seems to work with yum, however, one
# of the other mirror sites, http://archive.kernel.org/centos-vault does
RUN rm /etc/yum.repos.d/*
ADD CentOS-Base.repo /etc/yum.repos.d/
ADD CentOS-Debuginfo.repo /etc/yum.repos.d/
ADD CentOS-Sources.repo /etc/yum.repos.d/
ADD CentOS-fasttrack.repo /etc/yum.repos.d/
ADD libselinux.repo /etc/yum.repos.d/

RUN yum clean all -y
#commented out - likely unnecessary?
#RUN yum makecache -y
RUN yum update -y

RUN yum install -y freetype libSM libXi libXrender libXrandr \
	libXfixes libXcursor libXinerama fontconfig \
	libxslt xauth xorg-x11-server-Xvfb dbus-x11 \
	tkinter xterm perl
#separating for debugging
#try w/o this package, as is not working :(
#RUN yum install -y ImageMagick-c++

# setup all required env variables
ARG CASA_RELEASE
ENV CASA_RELEASE=${CASA_RELEASE}
ENV PATH /usr/local/sbin:/usr/local/bin:/usr/sbin:/usr/bin:/sbin:/bin:/opt/casa/bin

# untar casa databundle to container
ADD ${CASA_RELEASE}.tar.gz /opt/

# patch in fix for get_user.py
ARG PYTHON
ENV PYTHON=${PYTHON}
RUN rm -f /opt/${CASA_RELEASE}/lib64/${PYTHON}/get_user.py
ADD get_user.py /opt/${CASA_RELEASE}/lib64/${PYTHON}/get_user.py

# chown because the untarred casa has wrong owner/group
RUN chown -R root:root /opt/${CASA_RELEASE} && ln -s /opt/${CASA_RELEASE} /opt/casa

RUN yum install -y sssd-client acl which

# Allow runtime symlink creation to the casa-data-repository
RUN mkdir -p /arc/projects/casa-data-repository
RUN rm -rf /opt/${CASA_RELEASE}/data && \
    ln -s /arc/projects/casa-data-repository/ /opt/${CASA_RELEASE}/data
RUN chmod 777 /opt/${CASA_RELEASE}
RUN rm -rf /arc

RUN mkdir /skaha
ADD startup.sh /skaha/

# generate missing dbus uuid (issue #47)
RUN dbus-uuidgen --ensure

ADD nsswitch.conf /etc/

RUN chmod -R a+rwx /skaha

#HK updated below
#ENTRYPOINT [ "/skaha/startup.sh" ]
CMD [ "/skaha/init.sh" ]
