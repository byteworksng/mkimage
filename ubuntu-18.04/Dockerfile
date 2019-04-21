FROM ubuntu:18.04

ARG PHP_VERSION

ENV ARTIFACTS_PATH="/artifacts" \
	BUILD_HOME="/build" \
	DEBIAN_FRONTEND=noninteractive

RUN mkdir -p $ARTIFACTS_PATH $BUILD_HOME
WORKDIR $BUILD_HOME

RUN echo exit 101 > /usr/sbin/policy-rc.d \
	&& chmod +x /usr/sbin/policy-rc.d \
	&& echo "force-unsafe-io" > /etc/dpkg/dpkg.cfg.d/02apt-speedup \
    && apt update -y \
	&& apt upgrade -y -q \
	&& apt install -y -q --no-install-recommends \
		software-properties-common \
		ca-certificates \
		apt-transport-https \
		apt-utils \
	&& LANG=C.UTF-8 apt-add-repository -y ppa:ondrej/php \
	&& apt update -y

RUN apt install -y --no-install-recommends \
		php$PHP_VERSION-common \
		php$PHP_VERSION-cli \
		php$PHP_VERSION-xml \
		php$PHP_VERSION-dev \
		git \
		curl \
		sudo \
		gnupg \
		gcc \
		make \
		re2c \
		autoconf \
		automake \
		libpcre3-dev \
		tree \
		check \
		libssl-dev \
	&& echo '%adm ALL=(ALL) NOPASSWD:ALL' >> /etc/sudoers

RUN mkdir -p $ARTIFACTS_PATH/usr/lib/php/`php-config --phpapi` \
	&& mkdir -p $ARTIFACTS_PATH/etc/php/$PHP_VERSION/mods-available \
	&& mkdir -p $ARTIFACTS_PATH/usr/local/bin \
	&& mkdir -p $ARTIFACTS_PATH/usr/local/lib

# Zephir Parser
RUN git clone --depth=1 -q https://github.com/phalcon/php-zephir-parser.git -b master $BUILD_HOME/php-zephir-parser \
	&& cd $BUILD_HOME/php-zephir-parser \
	&& phpize \
    && ./configure \
    && make -j"$(getconf _NPROCESSORS_ONLN)" \
    && make install \
	&& cp $BUILD_HOME/php-zephir-parser/modules/zephir_parser.so $ARTIFACTS_PATH/usr/lib/php/`php-config --phpapi`/zephir_parser.so \
	&& echo "[Zephir]" >> $ARTIFACTS_PATH/etc/php/$PHP_VERSION/mods-available/zephir_parser.ini \
    && echo "extension=zephir_parser.so" >> $ARTIFACTS_PATH/etc/php/$PHP_VERSION/mods-available/zephir_parser.ini


# PSR
RUN git clone --depth=1 -q https://github.com/jbboehr/php-psr.git -b master $BUILD_HOME/php-psr \
	&& cd $BUILD_HOME/php-psr \
	&& phpize \
    && ./configure \
    && make -j"$(getconf _NPROCESSORS_ONLN)" \
    && make install \
	&& cp $BUILD_HOME/php-psr/modules/psr.so $ARTIFACTS_PATH/usr/lib/php/`php-config --phpapi`/psr.so \
	&& echo "[Psr]" >> $ARTIFACTS_PATH/etc/php/$PHP_VERSION/mods-available/psr.ini \
    && echo "extension=psr.so" >> $ARTIFACTS_PATH/etc/php/$PHP_VERSION/mods-available/psr.ini


# Phalcon
RUN git clone --depth=1 -q https://github.com/phalcon/cphalcon.git -b 4.0.x $BUILD_HOME/cphalcon \
	&& cd $BUILD_HOME/cphalcon/build \
	&& ./install --phpize /usr/bin/phpize$PHP_VERSION --php-config /usr/bin/php-config$PHP_VERSION \
	&& cp `php-config --extension-dir`/phalcon.so $ARTIFACTS_PATH/usr/lib/php/`php-config --phpapi`/phalcon.so \
	&& cp $BUILD_HOME/cphalcon/tests/_ci/phalcon.ini $ARTIFACTS_PATH/etc/php/$PHP_VERSION/mods-available/50-phalcon.ini

# Aerospike
RUN git clone --depth=1 -q https://github.com/aerospike/aerospike-client-php -b master $BUILD_HOME/aerospike-client-php \
	&& cd $BUILD_HOME/aerospike-client-php/src \
	&& chmod +x build.sh \
	&& ./build.sh --loglevel OFF \
	&& make install \
	&& cp $BUILD_HOME/aerospike-client-php/src/modules/aerospike.so $ARTIFACTS_PATH/usr/lib/php/`php-config --phpapi`/aerospike.so \
	&& echo "[Aerospike]" >> $ARTIFACTS_PATH/etc/php/$PHP_VERSION/mods-available/aerospike.ini \
	&& echo "; Note: To use aerospike extension you have to install libssl-dev package" >> $ARTIFACTS_PATH/etc/php/$PHP_VERSION/mods-available/aerospike.ini \
	&& echo "extension=aerospike.so" >> $ARTIFACTS_PATH/etc/php/$PHP_VERSION/mods-available/aerospike.ini \
	&& echo "aerospike.udf.lua_system_path=/usr/local/aerospike/lua" >> $ARTIFACTS_PATH/etc/php/$PHP_VERSION/mods-available/aerospike.ini \
	&& echo "aerospike.udf.lua_user_path=/usr/local/aerospike/usr-lua" >> $ARTIFACTS_PATH/etc/php/$PHP_VERSION/mods-available/aerospike.ini

# PHP HandlerSocket
RUN git clone --depth=1 -q https://github.com/tony2001/php-ext-handlersocketi.git -b badoo-7.0 $BUILD_HOME/php-ext-handlersocketi \
	&& cd $BUILD_HOME/php-ext-handlersocketi \
	&& phpize \
	&& ./configure \
	&& make -j"$(getconf _NPROCESSORS_ONLN)" \
	&& make install \
	&& cp $BUILD_HOME/php-ext-handlersocketi/modules/handlersocketi.so $ARTIFACTS_PATH/usr/lib/php/`php-config --phpapi`/handlersocketi.so \
	&& echo "extension=handlersocketi.so" >> $ARTIFACTS_PATH/etc/php/$PHP_VERSION/mods-available/handlersocketi.ini

# Pinba
RUN git clone --depth=1 -q https://github.com/tony2001/pinba_extension.git $BUILD_HOME/pinba_extension \
	&& cd $BUILD_HOME/pinba_extension \
	&& phpize \
	&& ./configure \
	&& make -j"$(getconf _NPROCESSORS_ONLN)" \
	&& make install \
	&& cp $BUILD_HOME/pinba_extension/modules/pinba.so $ARTIFACTS_PATH/usr/lib/php/`php-config --phpapi`/pinba.so \
	&& echo "extension=pinba.so" >> $ARTIFACTS_PATH/etc/php/$PHP_VERSION/mods-available/pinba.ini

# Weakref - not compatible with PHP7.3 yet. Diabled for now
#RUN git clone --depth=1 -q https://github.com/colder/php-weakref -b master $BUILD_HOME/php-weakref \
#	&& cd $BUILD_HOME/php-weakref \
#	&& phpize \
#	&& ./configure \
#	&& make -j"$(getconf _NPROCESSORS_ONLN)" \
#	&& make install \
#	&& cp $BUILD_HOME/php-weakref/modules/weakref.so $ARTIFACTS_PATH/usr/lib/php/`php-config --phpapi`/weakref.so \
#	&& echo "extension=weakref.so" >> $ARTIFACTS_PATH/etc/php/$PHP_VERSION/mods-available/weakref.ini

RUN tree -a $ARTIFACTS_PATH

RUN SUDO_FORCE_REMOVE=yes apt remove -y \
		php$PHP_VERSION-common \
		php$PHP_VERSION-cli \
		php$PHP_VERSION-xml \
		php$PHP_VERSION-dev \
		git \
		curl \
		sudo \
		gcc \
		make \
		re2c \
		autoconf \
		automake \
		libpcre3-dev \
		tree \
		check \
		libssl-dev \
	&& apt autoremove -y \
	&& apt autoclean -y \
	&& apt clean -y \
	&& rm -rf \
		/var/lib/apt/lists/* \
		/tmp/* \
		/var/tmp/* \
		/etc/php \
		/usr/lib/php \
		$BUILD_HOME

VOLUME ['$ARTIFACTS_PATH']
