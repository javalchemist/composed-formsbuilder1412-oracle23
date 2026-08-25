# DESCRIPTION


# Copyright (c) 2026 javalchemist
#
# Licensed under the Non-AI Apache 2.0 License 

# -------------
# it is used to create image with Forms Builder 14.1.2

# REQUIRED FILES
# -------------
# 1. a base image with oracle linux 9 + jdk21
#
# 2. binary installation package for Form Builder Standalone 14.1.2.0.0 . 
# Availlable from edelivery.oracle.com when searched for
# "Oracle Fusion Middleware 14c (14.1.2.0.0) Forms and Reports for Linux x86-64 for (Linux x86-64), 1.3 GB"
# The file available for download is "V1045121-01.zip" which unzipped gives "fmw_14.1.2.0.0_fr_linux64.bin". This is the binary file that will be used.

# 3. a responseFile for silent installation
#


FROM oracle/jdk:21-ol9 AS base_image

# create user:group = oracle:oracle
RUN groupadd oracle && \
   useradd -g oracle oracle && \
   passwd -d oracle;

# update OS + required packages
RUN  dnf install \
   binutils-2.35.2-42.0.1.el9.x86_64  \
	gcc-11.3.1-4.3.0.4.el9.x86_64 \
	gcc-c++-11.3.1-4.3.0.4.el9.x86_64 \
	glibc-2.34-100.0.1.el9_4.2.x86_64 \
	glibc-devel-2.34-100.0.1.el9_4.2.x86_64 \
	libaio-0.3.111-13.el9.x86_64 \
	libaio-devel-0.3.111-13.el9.x86_64 \
	libgcc-11.4.1-3.0.1.el9.x86_64 \
	libstdc++-11.3.1-4.3.0.4.el9.x86_64 \
	libstdc++-devel-11.3.1-4.3.0.4.el9.x86_64 \
	libnsl-2.34-100.0.1.el9_4.2.x86_64 \
	sysstat-12.5.4-7.0.1.el9.x86_64 \
	motif-2.3.4-28.el9.x86_64 \
	motif-devel-2.3.4-28.el9.x86_64 \
	openssl-3.0.7-27.0.3.el9.x86_64 \
	make-4.3-7.el9.x86_64 \
	xorg-x11-utils-7.5-40.el9.x86_64 \
	ksh-1.0.0~beta.1-3.0.1.el9.x86_64 \
	libcap-2.48-9.el9_2.x86_64 \
   xorg-x11-xauth \
   && dnf upgrade -y && dnf clean all \
   && find /var/log -type f -exec sh -c 'cat /dev/null > "$1"' _ {} \; \
   && rm -rf /var/lib/dnf/history*

# starts Forms Builder installation 
RUN mkdir -p /oracle && chown oracle:oracle /oracle
WORKDIR /oracle
USER oracle
# create central inventory location
RUN mkdir -p Oracle && \
    printf 'inventory_loc=/oracle/Oracle/oraInventory\ninst_group=oracle\n' \
        > /oracle/Oracle/oraInst.loc

# mount bind volume with installation package
# start silent installation

RUN --mount=type=bind,source=installer,target=/tmp/installer/ \
  /tmp/installer/fmw_14.1.2.0.0_fr_linux64.bin -silent \ 
  -responseFile /tmp/installer/responseFile \
  -logLevel Fine -ignoreSysPrereqs -invPtrLoc /oracle/Oracle/oraInst.loc \
   && find /oracle/Oracle -type f  -name '*.log' -delete

ENV ORACLE_HOME=/oracle/Oracle/Middleware/Oracle_Home
# start config wizard
RUN $ORACLE_HOME/forms/common/bin/config_builder.sh \
       autoconfig formBuilderInstance=$ORACLE_HOME/formsInst1 \
   && find /oracle/Oracle -type f  -name '*.log' -delete

# final form
FROM base_image AS final_image
COPY --from=base_image /oracle /oracle

WORKDIR /oracle
USER oracle
ARG DISPLAY=":0"
ENV DISPLAY=${DISPLAY}
ARG ORACLE_HOME=/oracle/Oracle/Middleware/Oracle_Home

# connect string to database
RUN cat << 'EOF1' >> /${ORACLE_HOME}/formsInst1/tnsnames.ora && \
    cat << 'EOF2' >> /${ORACLE_HOME}/network/admin/tnsnames.ora
fb14db =
(DESCRIPTION =
 (ADDRESS =
   (PROTOCOL = TCP)
   (HOST = db-fb1412-platform)
   (PORT = 1521))
 (CONNECT_DATA =
   (SERVER = DEDICATED)
   (SERVICE_NAME = freepdb1)))
EOF1
fb14db =
(DESCRIPTION =
 (ADDRESS =
   (PROTOCOL = TCP)
   (HOST = db-fb1412-platform)
   (PORT = 1521))
 (CONNECT_DATA =
   (SERVER = DEDICATED)
   (SERVICE_NAME = freepdb1)))
EOF2

ENTRYPOINT ["/bin/sh", "-c", "exec ${ORACLE_HOME}/formsInst1/bin/frmbld.sh \"$@\"", "--"]




