FROM alpine:3.12

# ensure local haxe is preferred over distribution haxe
ENV PATH /usr/local/bin:$PATH
ENV NEKO_VERSION 2.3.0
ENV HAXE_VERSION 4.2.5
ENV HAXE_STD_PATH /usr/local/share/haxe/std
ENV NODE_VERSION 20.11.1
ENV NODE_OPTIONS=--openssl-legacy-provider

# install ca-certificates so that HTTPS works consistently
# the other runtime dependencies are installed later
RUN apk add --no-cache ca-certificates

RUN set -ex \
	&& apk add --no-cache \
		libressl \
		tar \
		git \
		openssh \
		coreutils \
	 \
	&& wget -O neko.tar.gz "https://github.com/HaxeFoundation/neko/archive/v2-3-0/neko-2.3.0.tar.gz" \
	&& echo "850e7e317bdaf24ed652efeff89c1cb21380ca19f20e68a296c84f6bad4ee995 *neko.tar.gz" | sha256sum -c - \
	&& mkdir -p /usr/src/neko \
	&& tar -xC /usr/src/neko --strip-components=1 -f neko.tar.gz \
	&& rm neko.tar.gz \
	&& apk add --no-cache --virtual .neko-build-deps \
		apache2-dev \
		cmake \
		gc-dev \
		gcc \
		gtk+2.0-dev \
		libc-dev \
		linux-headers \
		mariadb-dev \
		mbedtls-dev \
		ninja \
		sqlite-dev \
	&& cd /usr/src/neko \
	&& cmake -GNinja -DNEKO_JIT_DISABLE=ON -DRELOCATABLE=OFF -DRUN_LDCONFIG=OFF . \
	&& ninja \
	&& ninja install \
	\
	&& git clone --recursive --depth 1 --branch 4.2.5 --single-branch "https://github.com/HaxeFoundation/haxe.git" /usr/src/haxe \
	&& cd /usr/src/haxe \
	&& mkdir -p $HAXE_STD_PATH \
	&& cp -r std/* $HAXE_STD_PATH \
	&& apk add --no-cache --virtual .haxe-build-deps \
		pcre-dev \
		zlib-dev \
		mbedtls-dev \
		make \
		\
		opam \
		aspcud \
		m4 \
		unzip \
		patch \
		musl-dev \
		perl-string-shellquote \
		perl-ipc-system-simple \
		\
		ocaml-compiler-libs \
		ocaml-ocamldoc \
		\
	\
	&& opam init --disable-sandboxing \
	&& eval $(opam env) \
	&& opam pin add haxe . --no-action \
	&& opam install haxe --deps-only --yes \
	&& make \
	&& eval $(opam env --revert) \
	\
	&& mkdir -p /usr/local/bin \
	&& cp haxe haxelib /usr/local/bin \
	&& mkdir -p /haxelib \
	&& cd / && haxelib setup /haxelib \
	\
	&& runDeps="$( \
		scanelf --needed --nobanner --recursive /usr/local \
			| awk '{ gsub(/,/, "\nso:", $2); print "so:" $2 }' \
			| sort -u \
			| xargs -r apk info --installed \
			| sort -u \
	)" \
	&& apk add --virtual .haxe-rundeps $runDeps \
	&& apk del .neko-build-deps .haxe-build-deps \
	&& rm -rf ~/.opam \
	&& rm -rf /usr/src/neko /usr/src/haxe
CMD ["haxe"]

# install nodejs
RUN addgroup -g 1000 node \
		&& adduser -u 1000 -G node -s /bin/sh -D node \
		&& apk add --no-cache \
				libstdc++ \
		&& apk add --no-cache --virtual .build-deps \
				curl \
		&& ARCH= OPENSSL_ARCH='linux*' && alpineArch="$(apk --print-arch)" \
			&& case "${alpineArch##*-}" in \
				x86_64) ARCH='x64' CHECKSUM="5da733c21c3b51193a4fe9fc5be6cfa9a694d13b8d766eb02dbe4b8996547050" OPENSSL_ARCH=linux-x86_64;; \
				x86) OPENSSL_ARCH=linux-elf;; \
				aarch64) OPENSSL_ARCH=linux-aarch64;; \
				arm*) OPENSSL_ARCH=linux-armv4;; \
				ppc64le) OPENSSL_ARCH=linux-ppc64le;; \
				s390x) OPENSSL_ARCH=linux-s390x;; \
				*) ;; \
			esac \
	&& if [ -n "${CHECKSUM}" ]; then \
		set -eu; \
		curl -fsSLO --compressed "https://unofficial-builds.nodejs.org/download/release/v$NODE_VERSION/node-v$NODE_VERSION-linux-$ARCH-musl.tar.xz"; \
		echo "$CHECKSUM	node-v$NODE_VERSION-linux-$ARCH-musl.tar.xz" | sha256sum -c - \
			&& tar -xJf "node-v$NODE_VERSION-linux-$ARCH-musl.tar.xz" -C /usr/local --strip-components=1 --no-same-owner \
			&& ln -s /usr/local/bin/node /usr/local/bin/nodejs; \
	else \
		echo "Building from source" \
		# backup build
		&& apk add --no-cache --virtual .build-deps-full \
				binutils-gold \
				g++ \
				gcc \
				gnupg \
				libgcc \
				linux-headers \
				make \
				python3 \
		# use pre-existing gpg directory, see https://github.com/nodejs/docker-node/pull/1895#issuecomment-1550389150
		&& export GNUPGHOME="$(mktemp -d)" \
		# gpg keys listed at https://github.com/nodejs/node#release-keys
		&& for key in \
			4ED778F539E3634C779C87C6D7062848A1AB005C \
			141F07595B7B3FFE74309A937405533BE57C7D57 \
			74F12602B6F1C4E913FAA37AD3A89613643B6201 \
			DD792F5973C6DE52C432CBDAC77ABFA00DDBF2B7 \
			61FC681DFB92A079F1685E77973F295594EC4689 \
			8FCCA13FEF1D0C2E91008E09770F7A9A5AE15600 \
			C4F0DFFF4E8C1A8236409D08E73BC641CC11F4C8 \
			890C08DB8579162FEE0DF9DB8BEAB4DFCF555EF4 \
			C82FA3AE1CBEDC6BE46B9360C43CEC45C17AB93C \
			108F52B48DB57BB0CC439B2997B01419BD92F80A \
			A363A499291CBBC940DD62E41F10027AF002F8B0 \
		; do \
			gpg --batch --keyserver hkps://keys.openpgp.org --recv-keys "$key" || \
			gpg --batch --keyserver keyserver.ubuntu.com --recv-keys "$key" ; \
		done \
		&& curl -fsSLO --compressed "https://nodejs.org/dist/v$NODE_VERSION/node-v$NODE_VERSION.tar.xz" \
		&& curl -fsSLO --compressed "https://nodejs.org/dist/v$NODE_VERSION/SHASUMS256.txt.asc" \
		&& gpg --batch --decrypt --output SHASUMS256.txt SHASUMS256.txt.asc \
		&& gpgconf --kill all \
		&& rm -rf "$GNUPGHOME" \
		&& grep " node-v$NODE_VERSION.tar.xz\$" SHASUMS256.txt | sha256sum -c - \
		&& tar -xf "node-v$NODE_VERSION.tar.xz" \
		&& cd "node-v$NODE_VERSION" \
		&& ./configure \
		&& make -j$(getconf _NPROCESSORS_ONLN) V= \
		&& make install \
		&& apk del .build-deps-full \
		&& cd .. \
		&& rm -Rf "node-v$NODE_VERSION" \
		&& rm "node-v$NODE_VERSION.tar.xz" SHASUMS256.txt.asc SHASUMS256.txt; \
	fi \
	&& rm -f "node-v$NODE_VERSION-linux-$ARCH-musl.tar.xz" \
	# Remove unused OpenSSL headers to save ~34MB. See this NodeJS issue: https://github.com/nodejs/node/issues/46451
	&& find /usr/local/include/node/openssl/archs -mindepth 1 -maxdepth 1 ! -name "$OPENSSL_ARCH" -exec rm -rf {} \; \
	&& apk del .build-deps \
	# smoke tests
	&& node --version \
	&& npm --version
