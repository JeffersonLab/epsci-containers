#--------------------------------------------------------------------------
#
#
# To build and push the container:
#
# docker build -f Dockerfile -t cernlib:2005 .
#
# docker tag cernlib:2005 jeffersonlab/cernlib:2005
#
# docker push jeffersonlab/cernlib:2005

FROM markito3/gluex_docker_devel:latest
LABEL maintainer "David Lawrence <davidl@jlab.org>"

RUN yum install -y \
	libXt \
	openmotif \
	xterm \
	libgfortran \
	gcc-gfortran

ADD cernlib_2005-ctr.tgz /cernlib
ADD cernlib.csh /etc/profile.d
ADD cernlib.sh /etc/profile.d

RUN mkdir -p /container/image
ADD Dockerfile /container/image
ADD dsh /container/image
ADD cernlib.csh /container/image
ADD cernlib.sh /container/image

# This cernlib was originally built to go in /cern/pro so make links
RUN   ln -s /cernlib /cern \
	&& ln -s 2005 /cernlib/pro

# Extract the dsh script with:
# docker run --rm -it cernlib:2005 cat /container/image/dsh | tr -d "\r" > dsh

ENV DISPLAY=host.docker.internal:0
ENV PATH=/cernlib/2005/bin:/usr/local/sbin:/usr/local/bin:/usr/sbin:/usr/bin:/sbin:/bin
CMD bash
