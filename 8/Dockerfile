# 基于Debian的Openjdk镜像,去除源码文件和演示样例.
FROM debian:buster-slim

MAINTAINER zooltech@qq.com

ENV	LANG="C.UTF-8" JAVA_HOME="/usr/local/openjdk-8"
ENV	PATH="$JAVA_HOME/bin:$PATH"

RUN	set -eux;\
# java install params
	JAVA_VERSION_1=8u265;\
	JAVA_VERSION_2=b01;\
	JAVA_DOWNLOAD_URL=https://github.com/AdoptOpenJDK/openjdk8-upstream-binaries/releases/download/jdk${JAVA_VERSION_1}-${JAVA_VERSION_2}/OpenJDK8U-jdk_x64_linux_${JAVA_VERSION_1}${JAVA_VERSION_2}.tar.gz;\
# init
	apt-get -q update && apt-get install -yq --no-install-recommends busybox ca-certificates p11-kit && rm -rf /var/lib/apt/lists/*;\
# busybox soft link
	for cmdStr in free ip less nc netstat nslookup ping ps tail top tracerout vi watch wget;do ln -s busybox /bin/${cmdStr};done;\
# download openjdk
	busybox wget -O openjdk.tgz "${JAVA_DOWNLOAD_URL}" -q;\
	mkdir -p "$JAVA_HOME";\
	tar --extract --file openjdk.tgz --directory "$JAVA_HOME" --strip-components 1 --no-same-owner;\
	rm -rf openjdk.tgz* "$JAVA_HOME/src.zip" "$JAVA_HOME/demo" && rm -rf "$JAVA_HOME/sample";\
# update "cacerts" bundle to use Debian's CA certificate
	mkdir -p /etc/ca-certificates/update.d;\
	{\
		echo '#!/usr/bin/env bash';\
		echo 'set -Eeuo pipefail';\
		echo 'if ! [ -d "$JAVA_HOME" ]; then echo >&2 "error: missing JAVA_HOME environment variable"; exit 1; fi';\
# 8-jdk uses "$JAVA_HOME/jre/lib/security/cacerts" and 8-jre and 11+ uses "$JAVA_HOME/lib/security/cacerts" directly (no "jre" directory)
		echo 'cacertsFile=; for f in "$JAVA_HOME/lib/security/cacerts" "$JAVA_HOME/jre/lib/security/cacerts"; do if [ -e "$f" ]; then cacertsFile="$f"; break; fi; done';\
		echo 'if [ -z "$cacertsFile" ] || ! [ -f "$cacertsFile" ]; then echo >&2 "error: failed to find cacerts file in $JAVA_HOME"; exit 1; fi';\
		echo 'trust extract --overwrite --format=java-cacerts --filter=ca-anchors --purpose=server-auth "$cacertsFile"';\
	} > /etc/ca-certificates/update.d/docker-openjdk;\
	chmod +x /etc/ca-certificates/update.d/docker-openjdk;\
	/etc/ca-certificates/update.d/docker-openjdk;\
# https://github.com/docker-library/openjdk/issues/331#issuecomment-498834472
	find "$JAVA_HOME/lib" -name '*.so' -exec dirname '{}' ';' | sort -u > /etc/ld.so.conf.d/docker-openjdk.conf;\
	ldconfig;\
#shell config
#	echo "alias ls='ls --color=auto'\nalias ll='ls -lA'" > /root/.bashrc;\
# basic smoke test
	java -version
