#--------------------------------------------------------------------------
#
#
# To build and push the container:
#
# docker build -f Dockerfile -t cernlib:2023 .
#
# docker tag cernlib:2023 jeffersonlab/cernlib:2023
#
# docker push jeffersonlab/cernlib:2023

FROM almalinux:9.2-20230718
LABEL maintainer "David Lawrence <davidl@jlab.org>"


# The following addresses issues trying to install system packages
# while on the JLab network which screws up the certificate verification
# causing curl downloads to fail.
RUN echo "sslverify=0" >> /etc/dnf/dnf.conf

# Install most system packages
RUN     dnf groupinstall -y "Development Tools" \
     && dnf install -y \
	libXt \
	openmotif-devel \
	xterm \
	libgfortran \
	gcc-gfortran \
	cmake \
	openssl-devel \
	libnsl2 \
	libX11-devel libXt-devel libXaw-devel freetype-devel
	

# Default almalinux9 does not seem to have the devel versions of these
# so we grab the RPMs manually and install them (echhh!)
RUN   curl --insecure https://repo.almalinux.org/almalinux/9/CRB/x86_64/os/Packages/libnsl2-devel-2.0.0-1.el9.x86_64.rpm -o libnsl2-devel-2.0.0-1.el9.x86_64.rpm \
   && curl --insecure https://repo.almalinux.org/almalinux/9/CRB/x86_64/os/Packages/libtirpc-devel-1.3.3-1.el9.x86_64.rpm -o libtirpc-devel-1.3.3-1.el9.x86_64.rpm \
   && dnf install -y libnsl2-devel-2.0.0-1.el9.x86_64.rpm libtirpc-devel-1.3.3-1.el9.x86_64.rpm \
   && rm -f libnsl2-devel-2.0.0-1.el9.x86_64.rpm libtirpc-devel-1.3.3-1.el9.x86_64.rpm


# There is a bug in the source (paw_motif/hmotif/scan.h) where it includes
# Xbae/Matrix.h when the file should be xbae/matrix.h. (Possibly from MacOS
# which uses a case-insensitive filesystem by default?).
# To circumvent this, add a symlink.
RUN cd /tmp \
  && mkdir -p cernlib-2023.08.14.0/paw_motif/Xbae \
  && ln -s ../xbae/matrix.h cernlib-2023.08.14.0/paw_motif/Xbae/Matrix.h

# For whatever reason they add a "-gfortran" suffix to all of the
# binaries and do not appear to give any cmake mechanism to change
# this. The sed command below modifies CMakeLists.txt to fix this.
#
# n.b. The ADD command below automatically unpacks the tarball.
#
ADD cernlib-2023.08.14.0.tar.gz /tmp
RUN cd /tmp \
	&& sed -i 's/set( COMPSUFFIX -gfortran)/#set( COMPSUFFIX -gfortran)/g' cernlib-2023.08.14.0/CMakeLists.txt \
	&& cmake -S cernlib-2023.08.14.0 -B build -DCMAKE_INSTALL_PREFIX=/cern/2023 \
	&& cmake --build build --target install -- -j8 \
	&& rm -rf /tmp/cernlib-2023.08.14.0 /tmp/build


RUN mkdir -p /container/image
ADD Dockerfile /container/image
ADD dsh /container/image
ADD cernlib.csh /container/image
ADD cernlib.sh /container/image

ADD cernlib.csh /etc/profile.d
ADD cernlib.sh /etc/profile.d

# Extract the dsh script with:
# docker run --rm -it cernlib:2005 cat /container/image/dsh | tr -d "\r" > dsh

ENV DISPLAY=host.docker.internal:0
ENV PATH=/cern/2023/bin:/usr/local/sbin:/usr/local/bin:/usr/sbin:/usr/bin:/sbin:/bin
CMD bash
