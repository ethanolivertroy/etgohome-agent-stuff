---
name: emasser-setup
description: Set up and configure the eMASSer CLI tool for interacting with the eMASS (Enterprise Mission Assurance Support Service) API. Use when installing emasser, configuring environment variables, or troubleshooting connection issues.
user-invocable: true
argument-hint: "[install|configure|test]"
---

# eMASSer Setup & Configuration

You are an expert in setting up and configuring the eMASSer CLI, a Ruby-based command-line interface for automating the eMASS (Enterprise Mission Assurance Support Service) REST API.

## What is eMASSer?

eMASSer is a CLI that automates routine eMASS business use-cases by leveraging the eMASS REST API. It supports GET, POST, PUT, and DELETE operations against all eMASS API endpoints.

Agent rule: use the `emasser` CLI first for eMASS operations. Agents should know how to query, read/export artifacts, upload files and scan results, update records, and verify changes with follow-up GET commands. Raw API calls are fallback only when the CLI cannot perform the task.

- GitHub: https://github.com/mitre/emasser
- API Docs: https://mitre.github.io/emass_client/docs/redoc/
- Interactive Swagger/Stoplight mock: https://mitre.github.io/emass_client/docs/renderer/

## Installation

### Via RubyGems (recommended)
```bash
gem install emasser
```

### Via GitHub source
```bash
git clone https://github.com/mitre/emasser.git emasser
cd emasser
gem build *.gemspec
gem install *.gem
```

### Via Docker
```bash
# Linux/Mac
docker run --rm -v $PWD/path-to-secrets:/data mitre/emasser:latest

# Windows
docker run --rm -v %cd%/path-to-secrets:/data mitre/emasser:latest
```

### Development mode (no gem build needed)
```bash
git clone https://github.com/mitre/emasser.git emasser
cd emasser
bundle install
bundle exec exe/emasser [command]
```

### Prerequisites
- Ruby 3.2 or greater
- git
- On Windows: cURL binary (libcurl.dll) — download from https://curl.se/windows/ and place in Ruby's `/bin` directory

## Environment Variables Configuration

Create a `.env` file in the directory where you run `emasser`. A `.env-example` file is provided in the emasser repository.

### Required Variables
```bash
export EMASSER_API_KEY='<The eMASS API key (api-key)>'
export EMASSER_HOST_URL='<The Full Qualified Domain Name (FQDN) for the eMASS server>'
export EMASSER_KEY_FILE_PATH='<Path to the eMASS key.pem private key file>'
export EMASSER_CERT_FILE_PATH='<Path to the eMASS client.pem certificate file>'
export EMASSER_KEY_FILE_PASSWORD='<Secret phrase used to protect the encryption key>'
```

### Conditionally Required
```bash
export EMASSER_USER_UID='<The eMASS User Unique Identifier (user-uid)>'
```

### Optional Variables
```bash
export EMASSER_CLIENT_SIDE_VALIDATION='true'   # Client side validation (default: true)
export EMASSER_VERIFY_SSL='true'               # Verify SSL (default: true)
export EMASSER_VERIFY_SSL_HOST='true'          # Verify host SSL (default: true)
export EMASSER_DEBUGGING='false'               # Set debugging (default: false)
export EMASSER_CLI_DISPLAY_NULL='true'         # Display null value fields (default: true)
export EMASSER_EPOCH_TO_DATETIME='false'       # Convert epoch to date/time (default: false)
export EMASSER_DOWNLOAD_DIR='eMASSerDownloads' # Directory for exported files
```

> **Note:** Authentication and authorization to an eMASS instance must be set up with the eMASS instance owner organization separately. See [DCSA eMASS](https://www.dcsa.mil/is/emass/) for access information.

## Test Connection

After configuration, verify the connection:
```bash
emasser get test connection
```

A successful response confirms the CLI can reach the configured server URL.

## Mock API Validation

The official OpenAPI spec includes a hosted Stoplight Prism mock server:

```bash
curl -H 'Prefer: code=200' \
  https://stoplight.io/mocks/mitre/emasser/32836028/api
```

For normal security headers, any values are accepted by the mock:

```bash
curl -H 'api-key: 123' -H 'user-uid: 123' \
  https://stoplight.io/mocks/mitre/emasser/32836028/api/systems
```

Use the mock to validate endpoint paths, query parameters, and JSON request bodies. The stock `emasser` CLI cannot directly target this hosted mock URL because it sets `base_path=/` and strips path components from `EMASSER_HOST_URL`; use direct mock API calls for OpenAPI validation and `emasser` against real eMASS instances.

## Common CLI Patterns

- Use `--parameterName` for boolean TRUE values
- Use `--no-parameterName` for boolean FALSE values
- All dates use Unix time format (e.g., `1499644800`)

## Getting Help

```bash
# List all available GET commands
emasser get help

# List all available POST commands
emasser post help

# List all available PUT commands
emasser put help

# List all available DELETE commands
emasser delete help

# Get help for a specific command
emasser get help artifacts
emasser get artifacts help export
```
