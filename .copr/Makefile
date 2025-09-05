# COPR Makefile for automated php-brotli builds
# Downloads php-ext-brotli from GitHub and builds PHP extension

# COPR provides outdir, set default for local testing
outdir ?= .

# php-ext-brotli version (latest release)
BROTLI_VERSION = 0.18.2
BROTLI_URL = https://github.com/kjdev/php-ext-brotli/archive/refs/tags/$(BROTLI_VERSION).tar.gz

# Default target for COPR
srpm:
	@echo "=== Starting php-brotli SRPM build ==="
	@echo "Output directory: $(outdir)"
	@echo "Working directory: $$(pwd)"
	
	# Install git if not available (for COPR environment)
	@if ! command -v git >/dev/null 2>&1; then \
		echo "=== Installing git ==="; \
		if command -v dnf >/dev/null 2>&1; then \
			dnf install -y git || exit 1; \
		elif command -v yum >/dev/null 2>&1; then \
			yum install -y git || exit 1; \
		else \
			echo "ERROR: No package manager found to install git"; \
			exit 1; \
		fi; \
	fi
	
	# Download php-ext-brotli source
	@echo "=== Downloading php-ext-brotli $(BROTLI_VERSION) ==="
	curl -L -o php-ext-brotli-$(BROTLI_VERSION).tar.gz $(BROTLI_URL) || exit 1
	
	# Extract and prepare source with bundled brotli
	@echo "=== Preparing source with bundled brotli ==="
	tar xzf php-ext-brotli-$(BROTLI_VERSION).tar.gz
	mv php-ext-brotli-$(BROTLI_VERSION) php-ext-brotli-temp
	cd php-ext-brotli-temp && \
	git init && \
	git remote add origin https://github.com/kjdev/php-ext-brotli.git && \
	git fetch --depth=1 origin $(BROTLI_VERSION) && \
	git reset --hard FETCH_HEAD && \
	git submodule update --init --recursive && \
	cd .. && \
	tar czf php-brotli-$(BROTLI_VERSION).tar.gz php-ext-brotli-temp/ --transform='s/php-ext-brotli-temp/php-ext-brotli-$(BROTLI_VERSION)/' || exit 1
	rm -rf php-ext-brotli-temp/
	
	# Generate spec file
	@echo "=== Generating spec file ==="
	@( \
	echo '# we dont want -z defs linker flag'; \
	echo '%undefine _strict_symbol_defs_build'; \
	echo ''; \
	echo '%global pecl_name   brotli'; \
	echo '%global with_zts    0%{!?_without_zts:%{?__ztsphp:1}}'; \
	echo '%global ini_name    40-%{pecl_name}.ini'; \
	echo ''; \
	echo 'Name:           php-%{pecl_name}'; \
	echo 'Version:        $(BROTLI_VERSION)'; \
	echo 'Release:        1%{?dist}'; \
	echo 'Summary:        Brotli Extension for PHP'; \
	echo ''; \
	echo 'License:        MIT'; \
	echo 'URL:            https://github.com/kjdev/php-ext-brotli'; \
	echo 'Source0:        php-brotli-%{version}.tar.gz'; \
	echo ''; \
	echo 'BuildRequires:  gcc'; \
	echo 'BuildRequires:  make'; \
	echo 'BuildRequires:  php-devel >= 7.0'; \
	echo ''; \
	echo 'Requires:       php(zend-abi) = %{php_zend_api}'; \
	echo 'Requires:       php(api) = %{php_core_api}'; \
	echo ''; \
	echo 'Provides:       bundled(brotli) = 1.1.0'; \
	echo ''; \
	echo '%description'; \
	echo 'This extension allows Brotli compression and decompression.'; \
	echo ''; \
	echo 'Brotli is a generic-purpose lossless compression algorithm that'; \
	echo 'compresses data using a combination of a modern variant of the LZ77'; \
	echo 'algorithm, Huffman coding and 2nd order context modeling, with a'; \
	echo 'compression ratio comparable to the best currently available'; \
	echo 'general-purpose compression methods.'; \
	echo ''; \
	echo '%package devel'; \
	echo 'Summary:        Development files for %{name}'; \
	echo 'Requires:       %{name}%{?_isa} = %{version}-%{release}'; \
	echo 'Requires:       php-devel%{?_isa}'; \
	echo ''; \
	echo '%description devel'; \
	echo 'These are the files needed to compile programs using %{name}.'; \
	echo ''; \
	echo '%prep'; \
	echo '%setup -q -n php-ext-brotli-%{version}'; \
	echo ''; \
	echo '# Sanity check'; \
	echo 'extver=$$(sed -n '"'"'/#define BROTLI_EXT_VERSION/{s/.* "//;s/".*$$//;p}'"'"' php_brotli.h)'; \
	echo 'if test "x$${extver}" != "x%{version}"; then'; \
	echo '   : Error: Upstream extension version is $${extver}, expecting %{version}.'; \
	echo '   exit 1'; \
	echo 'fi'; \
	echo ''; \
	echo '%if %{with_zts}'; \
	echo '# Duplicate for ZTS build'; \
	echo 'cp -pr . ../ZTS'; \
	echo '%endif'; \
	echo ''; \
	echo '%build'; \
	echo '# NTS build'; \
	echo '%{_bindir}/phpize'; \
	echo '%configure \'; \
	echo '    --with-php-config=%{_bindir}/php-config \'; \
	echo '    --with-libdir=%{_lib} \'; \
	echo '    --enable-brotli'; \
	echo ''; \
	echo 'make %{?_smp_mflags}'; \
	echo ''; \
	echo '%if %{with_zts}'; \
	echo '# ZTS build'; \
	echo 'cd ../ZTS'; \
	echo '%{_bindir}/zts-phpize'; \
	echo '%configure \'; \
	echo '    --with-php-config=%{_bindir}/zts-php-config \'; \
	echo '    --with-libdir=%{_lib} \'; \
	echo '    --enable-brotli'; \
	echo ''; \
	echo 'make %{?_smp_mflags}'; \
	echo '%endif'; \
	echo ''; \
	echo '%install'; \
	echo '# Install NTS extension'; \
	echo 'make install INSTALL_ROOT=%{buildroot}'; \
	echo ''; \
	echo '# Install config file'; \
	echo 'install -d %{buildroot}%{php_inidir}'; \
	echo 'cat > %{buildroot}%{php_inidir}/%{ini_name} << EOF'; \
	echo '; Enable brotli extension module'; \
	echo 'extension = %{pecl_name}.so'; \
	echo ''; \
	echo '; Configuration'; \
	echo ';brotli.output_compression = 0'; \
	echo ';brotli.output_compression_level = -1'; \
	echo 'EOF'; \
	echo ''; \
	echo '%if %{with_zts}'; \
	echo '# Install ZTS extension'; \
	echo 'cd ../ZTS'; \
	echo 'make install INSTALL_ROOT=%{buildroot}'; \
	echo ''; \
	echo '# Install ZTS config file'; \
	echo 'install -d %{buildroot}%{php_ztsinidir}'; \
	echo 'cp %{buildroot}%{php_inidir}/%{ini_name} \'; \
	echo '   %{buildroot}%{php_ztsinidir}/%{ini_name}'; \
	echo '%endif'; \
	echo ''; \
	echo '# Install headers for devel package'; \
	echo 'install -d %{buildroot}%{php_incldir}/ext/%{pecl_name}'; \
	echo 'cp -p php_brotli.h %{buildroot}%{php_incldir}/ext/%{pecl_name}/'; \
	echo ''; \
	echo '%check'; \
	echo '# Minimal load test for NTS extension'; \
	echo '%{__php} --no-php-ini \'; \
	echo '    --define extension=%{buildroot}%{php_extdir}/%{pecl_name}.so \'; \
	echo '    --modules | grep "^%{pecl_name}$$"'; \
	echo ''; \
	echo '# Upstream test suite for NTS extension'; \
	echo 'TEST_PHP_EXECUTABLE=%{__php} \'; \
	echo 'TEST_PHP_ARGS="-n -d extension=%{buildroot}%{php_extdir}/%{pecl_name}.so" \'; \
	echo 'REPORT_EXIT_STATUS=1 \'; \
	echo '%{__php} -n run-tests.php -q --show-diff'; \
	echo ''; \
	echo '%if %{with_zts}'; \
	echo '# Minimal load test for ZTS extension'; \
	echo '%{__ztsphp} --no-php-ini \'; \
	echo '    --define extension=%{buildroot}%{php_ztsextdir}/%{pecl_name}.so \'; \
	echo '    --modules | grep "^%{pecl_name}$$"'; \
	echo ''; \
	echo '# Upstream test suite for ZTS extension'; \
	echo 'cd ../ZTS'; \
	echo 'TEST_PHP_EXECUTABLE=%{__ztsphp} \'; \
	echo 'TEST_PHP_ARGS="-n -d extension=%{buildroot}%{php_ztsextdir}/%{pecl_name}.so" \'; \
	echo 'REPORT_EXIT_STATUS=1 \'; \
	echo '%{__ztsphp} -n run-tests.php -q --show-diff'; \
	echo '%endif'; \
	echo ''; \
	echo '%files'; \
	echo '%license LICENSE'; \
	echo '%doc README.md'; \
	echo '%config(noreplace) %{php_inidir}/%{ini_name}'; \
	echo '%{php_extdir}/%{pecl_name}.so'; \
	echo ''; \
	echo '%if %{with_zts}'; \
	echo '%config(noreplace) %{php_ztsinidir}/%{ini_name}'; \
	echo '%{php_ztsextdir}/%{pecl_name}.so'; \
	echo '%endif'; \
	echo ''; \
	echo '%files devel'; \
	echo '%{php_incldir}/ext/%{pecl_name}'; \
	echo ''; \
	echo '%changelog'; \
	echo "* $$(date '+%a %b %d %Y') COPR Builder - $(BROTLI_VERSION)-1"; \
	echo '- Automated build from GitHub release $(BROTLI_VERSION)'; \
	echo '- Built with bundled brotli library'; \
	echo '- Supports both NTS and ZTS builds'; \
	) > php-brotli.spec || exit 1
	
	@echo "Spec file generated"
	
	# Build SRPM
	@echo "=== Building SRPM ==="
	rpmbuild -bs \
		--define "_sourcedir $$(pwd)" \
		--define "_specdir $$(pwd)" \
		--define "_builddir $$(pwd)" \
		--define "_srcrpmdir $(outdir)" \
		--define "_rpmdir $$(pwd)" \
		php-brotli.spec || exit 1
	
	@echo "=== SRPM build complete ==="
	ls -la $(outdir)/*.src.rpm

clean:
	@rm -f *.tar.gz *.spec *.src.rpm
	@rm -rf php-ext-brotli-*/

.PHONY: srpm clean