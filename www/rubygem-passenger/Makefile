# Created by: Jacob Atzen <jatzen@gmail.com>
# $FreeBSD$

PORTNAME=	passenger
PORTVERSION=	5.1.12
PORTREVISION=	2
CATEGORIES=	www rubygems
MASTER_SITES=	RG
PKGNAMEPREFIX=	rubygem-

MAINTAINER=	osa@FreeBSD.org
COMMENT=	Modules for running Ruby on Rails and Rack applications

LICENSE=	MIT
LICENSE_FILE=	${WRKSRC}/LICENSE

BUILD_DEPENDS=	rake:devel/rubygem-rake

CPE_VENDOR=	phusion
CPE_PRODUCT=	passenger

FLAVORS=	apache nginx
FLAVOR?=	${FLAVORS:[1]}

apache_PKGNAMESUFFIX=	-apache
nginx_PKGNAMESUFFIX=	-nginx

OPTIONS_DEFINE=	DEBUG SYMLINK
OPTIONS_EXCLUDE=DOCS
SYMLINK_DESC=	Create passenger symlink

.if ${FLAVOR} == apache
USES+=		apache:2.2+
.endif

USE_RUBY=	yes
RAKE_BIN=	${LOCALBASE}/bin/rake
USES+=		cpe gem libtool python:env shebangfix ssl
SHEBANG_FILES=	\
	src/cxx_supportlib/vendor-copy/libuv/gyp_uv.py \
	dev/vagrant/nginx_start \
	dev/vagrant/provision.sh \
	dev/ci/run-tests-with-docker \
	dev/ci/tests/debian/run \
	dev/ci/tests/rpm/run \
	dev/ci/scripts/setup-host-natively.sh \
	dev/ci/scripts/docker-entrypoint-stage2.sh \
	dev/ci/scripts/inituidgid \
	dev/ci/scripts/debug-console-wrapper.sh \
	dev/ci/scripts/run-tests-natively-stage2.sh \
	dev/ci/scripts/docker-entrypoint.sh \
	dev/ci/run-tests-natively \
	dev/ci/setup-host

LIB_DEPENDS+=	libuv.so:devel/libuv \
		libcurl.so:ftp/curl \
		libunwind.so:devel/libunwind \
		libapr-1.so:devel/apr1

BUILD_DEPENDS+=	rubygem-rack16>=0:www/rubygem-rack16
RUN_DEPENDS:=	${BUILD_DEPENDS}

SUB_LIST+=	GEM_LIB_DIR=${GEM_LIB_DIR} RUBY=${RUBY}
SUB_FILES=	pkg-message

PLIST_FILES=	bin/passenger \
		bin/passenger-config \
		bin/passenger-install-apache2-module \
		bin/passenger-memory-stats \
		bin/passenger-status \
		bin/passenger-install-nginx-module

SYMLINK_PLIST_FILES=	${GEMS_DIR}/${PORTNAME}
SYMLINK_SUB_LIST=	PASSENGER_INSTALL_DIR="${PREFIX}/${GEMS_DIR}/${PORTNAME}"
SYMLINK_SUB_LIST_OFF=	PASSENGER_INSTALL_DIR="${PREFIX}/${GEM_LIB_DIR}"

pre-patch:
.if ${FLAVOR} == nginx
	@${ECHO_CMD}
	@${ECHO_CMD} "Do not forget compile www/nginx"
	@${ECHO_CMD} "with PASSENGER_MODULE support"
	@${ECHO_CMD}
.endif
	@${REINPLACE_CMD} \
		's!-Wall!!g; \
		s!gcc!${CC}!g; \
		s!g++!${CXX}!g; \
		s!#{PlatformInfo.debugging_cflags}!${CFLAGS}!g; \
		s!-O2!!g; \
		s! -feliminate-unused-debug-symbols -feliminate-unused-debug-types!!g; \
		155s!true!false!' \
		${WRKSRC}/build/basics.rb
	@${REINPLACE_CMD} '1s:python:python2:' \
		${WRKSRC}/src/cxx_supportlib/vendor-copy/libuv/gyp_uv.py

pre-patch-DEBUG-off:
	@${REINPLACE_CMD} \
		's!-DPASSENGER_DEBUG!-DNDEBUG!g' \
		${WRKSRC}/build/basics.rb

post-build:
.if ${FLAVOR} == apache
	(CC=${CC} CXX=${CXX} ${WRKSRC}/bin/passenger-install-apache2-module --auto)
.endif

.if ${FLAVOR} == nginx
	(cd ${WRKSRC} && CC=${CC} CXX=${CXX} ${RAKE_BIN} nginx)
.endif
	${FIND} ${WRKSRC} -name '*.o' -delete
	${FIND} ${WRKSRC} -name '*.bak' -delete

post-install:
	( cd ${WRKSRC} && \
	${COPYTREE_SHARE} buildout ${STAGEDIR}${PREFIX}/${GEMS_DIR}/${PORTNAME}-${PORTVERSION} )
	${CHMOD} +x ${STAGEDIR}${PREFIX}/${GEMS_DIR}/${PORTNAME}-${PORTVERSION}/buildout/support-binaries/*
	${STRIP_CMD} ${STAGEDIR}${PREFIX}/${GEM_LIB_DIR}/buildout/ruby/*/passenger_native_support.so
	${STRIP_CMD} ${STAGEDIR}${PREFIX}/${GEM_LIB_DIR}/buildout/support-binaries/PassengerAgent
	${FIND} ${STAGEDIR}${PREFIX}/${GEM_LIB_DIR} -type d -empty -delete
.if ${FLAVOR} == apache
	${STRIP_CMD} ${STAGEDIR}${PREFIX}/${GEM_LIB_DIR}/buildout/apache2/mod_passenger.so
	${MKDIR} ${STAGEDIR}${PREFIX}/${APACHEMODDIR}
	${INSTALL} -lrs ${STAGEDIR}${PREFIX}/${GEM_LIB_DIR}/buildout/apache2/mod_passenger.so ${STAGEDIR}${PREFIX}/${APACHEMODDIR}/mod_passenger.so
	@${ECHO} "${APACHEMODDIR}/mod_passenger.so" >> ${TMPPLIST}
.endif

post-install-SYMLINK-on:
	${LN} -s ${GEM_NAME} ${STAGEDIR}${PREFIX}/${GEMS_DIR}/${PORTNAME}

.include <bsd.port.mk>
