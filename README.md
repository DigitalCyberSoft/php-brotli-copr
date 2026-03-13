# PHP-Brotli COPR Build Repository

Automated COPR build configuration for [php-ext-brotli](https://github.com/kjdev/php-ext-brotli) - PHP extension for Brotli compression.

## Overview

This repository contains the COPR build configuration to automatically build RPM packages for the PHP Brotli extension from GitHub releases.

## Features

- **Automated Builds**: Builds directly from php-ext-brotli GitHub releases
- **PHP Version Support**: Builds for the system PHP version (7.0+)
- **NTS and ZTS Support**: Builds both non-thread-safe and thread-safe variants
- **Bundled libbrotli**: Includes bundled Brotli library (1.1.0) by default
- **Test Suite**: Runs upstream test suite during build

## COPR Repository

The built packages are available in COPR at:
```
https://copr.fedorainfracloud.org/coprs/reversejames/php-brotli/
```

### Installation

```bash
# Enable the COPR repository
sudo dnf copr enable reversejames/php-brotli

# Install php-brotli
sudo dnf install php-brotli

# For development headers
sudo dnf install php-brotli-devel

# Restart PHP-FPM if using it
sudo systemctl restart php-fpm
```

## Configuration

The extension is configured via `/etc/php.d/40-brotli.ini`:
```ini
; Enable brotli extension module
extension = brotli.so
```

## Verification

```bash
php -m | grep brotli
php -i | grep brotli
```

## Build Process

1. Downloads the php-ext-brotli source from GitHub release tag
2. Fetches bundled brotli library via git submodules
3. Validates version against `php_brotli.h`
4. Generates RPM spec file dynamically
5. Builds SRPM for COPR

## Local Testing

```bash
cd .copr
make srpm
```

## Dependencies

- **Build**: gcc, make, php-devel (>= 7.0)
- **Runtime**: php

## Extension Features

- Brotli compression and decompression
- Streaming support for large files
- Output compression handler
- Compatible with PHP 7.0 and later

## License

The build configuration is provided as-is. php-ext-brotli is licensed under the MIT License.
The bundled brotli library is licensed under the MIT License.

## Upstream

- php-ext-brotli GitHub: https://github.com/kjdev/php-ext-brotli
- Brotli: https://github.com/google/brotli
