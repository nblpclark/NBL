# NetBox Disco Config Plugin - Complete Package for GitHub

## Directory Structure

```
netbox-disco-config/
├── .github/
│   └── workflows/
│       └── tests.yml
├── docs/
│   ├── installation.md
│   ├── configuration.md
│   ├── api.md
│   └── examples.md
├── netbox_disco_config/
│   ├── __init__.py
│   ├── models.py
│   ├── forms.py
│   ├── views.py
│   ├── urls.py
│   ├── navigation.py
│   ├── generators.py
│   ├── tables.py
│   ├── filtersets.py
│   ├── api/
│   │   ├── __init__.py
│   │   ├── serializers.py
│   │   ├── views.py
│   │   └── urls.py
│   ├── migrations/
│   │   ├── __init__.py
│   │   └── 0001_initial.py
│   ├── templates/
│   │   └── netbox_disco_config/
│   │       ├── builder.html
│   │       ├── discoconfiguration.html
│   │       └── discoconfiguration_list.html
│   └── static/
│       └── netbox_disco_config/
│           ├── css/
│           │   └── styles.css
│           └── js/
│               └── builder.js
├── tests/
│   ├── __init__.py
│   ├── test_models.py
│   ├── test_views.py
│   └── test_api.py
├── .gitignore
├── CHANGELOG.md
├── CONTRIBUTING.md
├── LICENSE
├── MANIFEST.in
├── Makefile
├── QUICK_START.md
├── README.md
├── build.sh
├── deploy.sh
├── docker-compose.yml
├── install.sh
├── requirements-dev.txt
├── requirements.txt
├── setup.py
├── setup_dev.sh
├── test.sh
└── uninstall.sh
```

---

# File Contents

## README.md

```markdown
# NetBox Disco Config

[![Version](https://img.shields.io/pypi/v/netbox-disco-config.svg)](https://pypi.org/project/netbox-disco-config/)
[![License](https://img.shields.io/github/license/yourusername/netbox-disco-config.svg)](LICENSE)
[![Python](https://img.shields.io/pypi/pyversions/netbox-disco-config.svg)](https://pypi.org/project/netbox-disco-config/)
[![NetBox](https://img.shields.io/badge/NetBox-4.0+-blue.svg)](https://github.com/netbox-community/netbox)

A comprehensive NetBox plugin for building and managing network discovery configurations with multi-credential support and HashiCorp Vault integration. Generate configuration files for NetBox ORB agents and other discovery tools.

![NetBox Disco Config Screenshot](docs/images/screenshot.png)

## Features

### 🔐 Multi-Credential Management
- Define multiple credential sets organized by security zone
- Support for Production, Staging, DMZ, IoT, and Vendor zones
- Per-device credential overrides
- Flexible tagging system

### 🔒 HashiCorp Vault Integration
- Token authentication
- AppRole authentication for machine-to-machine
- Kubernetes authentication for K8s deployments
- Dynamic secret retrieval
- Automatic credential rotation support

### 🔍 Discovery Policy Configuration
- **Network Discovery**: ICMP/TCP/UDP scanning of CIDR ranges
- **Device Discovery**: SSH-based device interrogation
- Cron-based scheduling
- Multiple policies per configuration
- Per-policy credential assignment

### 📄 Configuration File Generation
- Automatic `.env` file generation
- YAML configuration file generation
- Download via web UI or REST API
- Environment variable or Vault mode

### 🌐 REST API
- Full CRUD operations for all resources
- Programmatic file generation endpoints
- Filtering and pagination
- OpenAPI/Swagger documentation

### 🎨 Web Interface
- User-friendly configuration builder
- Visual credential set management
- Policy editor with syntax validation
- Real-time configuration preview

## Installation

### Requirements

- NetBox 4.0.0 or later
- Python 3.10 or later
- PostgreSQL 12 or later

### Quick Install

```bash
# Install the plugin
pip install netbox-disco-config

# Add to NetBox configuration
echo "netbox_disco_config" >> /opt/netbox/netbox/netbox/configuration.py

# Run migrations
cd /opt/netbox/netbox
python manage.py migrate netbox_disco_config

# Collect static files
python manage.py collectstatic --no-input

# Restart NetBox
sudo systemctl restart netbox netbox-rq
```

### Configuration

Edit `/opt/netbox/netbox/netbox/configuration.py`:

```python
PLUGINS = [
    'netbox_disco_config',
]

PLUGINS_CONFIG = {
    'netbox_disco_config': {
        'enable_vault_integration': True,
        'max_credential_sets': 50,
        'default_diode_target': 'https://your-instance.netboxcloud.com/diode',
    }
}
```

## Quick Start

### 1. Create Your First Configuration

Navigate to **Plugins** → **Disco Configurations** → **Add**

Fill in:
- **Name**: `production_config`
- **Agent Name**: `prod_agent_01`
- **Diode Target**: `https://netbox.example.com/diode`
- **Diode Client ID**: Your client ID
- **Diode Client Secret**: Your client secret
- **Credential Mode**: Environment Variables or Vault

### 2. Add Credential Sets

Go to the **Credentials** tab and add credential sets:

```
Name: production_core
Security Zone: Production
Description: Production core network devices
Username: netops_prod
Password: ••••••••
Tags: production, core, critical
```

### 3. Configure Discovery Policies

**Network Discovery:**
```
Name: headquarters_network
Schedule: 0 */2 * * * (Every 2 hours)
Targets: 10.0.0.0/8, 192.168.0.0/16
```

**Device Discovery:**
```
Name: core_switches
Schedule: 0 */6 * * * (Every 6 hours)
Default Site: Headquarters
Credentials: production_core
Devices:
  - Driver: ios, Hostname: 10.0.1.1
  - Driver: iosxe, Hostname: 10.0.1.2
```

### 4. Generate Configuration Files

Go to the **Generate Files** tab:
- Download `.env` file
- Download `agent.yaml` file

### 5. Deploy

```bash
# Set permissions
chmod 600 .env

# Load environment variables
export $(cat .env | xargs)

# Run with Docker
docker run --net=host \
  -v $(pwd)/agent.yaml:/opt/orb/agent.yaml \
  --env-file .env \
  netboxlabs/orb-agent:latest run -c /opt/orb/agent.yaml
```

## Usage Examples

### Python/Django Shell

```python
from netbox_disco_config.models import DiscoConfiguration, CredentialSet

# Create configuration
config = DiscoConfiguration.objects.create(
    name='production_config',
    agent_name='prod_agent_01',
    diode_target='https://netbox.example.com/diode',
    diode_client_id='your_client_id',
    diode_client_secret='your_secret',
    credential_mode='env'
)

# Add credential sets
CredentialSet.objects.create(
    disco_config=config,
    name='production_core',
    description='Production core infrastructure',
    security_zone='production',
    username='netops_prod',
    password='SecurePassword123!',
    tags=['production', 'core']
)

# Generate files
from netbox_disco_config.generators import EnvGenerator, YAMLGenerator

env_gen = EnvGenerator(config)
yaml_gen = YAMLGenerator(config)

print(env_gen.generate())
print(yaml_gen.generate())
```

### REST API

```bash
# List configurations
curl -H "Authorization: Token YOUR_TOKEN" \
  http://netbox.example.com/api/plugins/disco-config/configurations/

# Get specific configuration
curl -H "Authorization: Token YOUR_TOKEN" \
  http://netbox.example.com/api/plugins/disco-config/configurations/1/

# Generate .env file
curl -H "Authorization: Token YOUR_TOKEN" \
  http://netbox.example.com/api/plugins/disco-config/configurations/1/generate_env/ \
  -o production.env

# Generate agent.yaml
curl -H "Authorization: Token YOUR_TOKEN" \
  http://netbox.example.com/api/plugins/disco-config/configurations/1/generate_yaml/ \
  -o agent.yaml
```

## HashiCorp Vault Integration

### Setup Vault

```bash
# Enable KV v2 secrets engine
vault secrets enable -path=secret kv-v2

# Create policy
vault policy write disco-config - <<EOF
path "secret/data/network/*" {
  capabilities = ["read", "list"]
}
EOF

# Store credentials
vault kv put secret/network/production/core \
  username="netops_prod" \
  password="SecurePassword123!" \
  enable_password="EnablePassword456!"

# Create AppRole
vault auth enable approle
vault write auth/approle/role/disco-config \
  token_policies="disco-config" \
  token_ttl=1h token_max_ttl=4h

# Get role credentials
vault read auth/approle/role/disco-config/role-id
vault write -f auth/approle/role/disco-config/secret-id
```

### Configure in NetBox

Switch configuration to Vault mode and add:
- Vault Address: `https://vault.example.com:8200`
- Auth Method: `approle`
- Role ID: `<from above>`
- Secret ID: `<from above>`

Update credential sets with Vault paths:
- Vault Path: `secret/data/network/production/core`
- Username Key: `username`
- Password Key: `password`

## Security Best Practices

### 1. Credential Storage
- ✅ Use HashiCorp Vault for production
- ✅ Never commit `.env` files to version control
- ✅ Set file permissions: `chmod 600 .env`
- ✅ Rotate credentials regularly

### 2. Credential Separation
- ✅ Separate by security zone (production, staging, DMZ)
- ✅ Use least privilege principle
- ✅ Different credentials per environment
- ✅ Audit access regularly

### 3. NetBox Access
- ✅ Restrict plugin access to authorized users
- ✅ Use NetBox's permission system
- ✅ Enable audit logging
- ✅ Regular security reviews

## Documentation

- [Installation Guide](docs/installation.md)
- [Configuration Guide](docs/configuration.md)
- [API Documentation](docs/api.md)
- [Usage Examples](docs/examples.md)

## API Reference

### Endpoints

- `GET /api/plugins/disco-config/configurations/` - List configurations
- `POST /api/plugins/disco-config/configurations/` - Create configuration
- `GET /api/plugins/disco-config/configurations/{id}/` - Get configuration
- `PUT /api/plugins/disco-config/configurations/{id}/` - Update configuration
- `DELETE /api/plugins/disco-config/configurations/{id}/` - Delete configuration
- `GET /api/plugins/disco-config/configurations/{id}/generate_env/` - Generate .env
- `GET /api/plugins/disco-config/configurations/{id}/generate_yaml/` - Generate YAML
- `GET /api/plugins/disco-config/credential-sets/` - List credential sets
- `GET /api/plugins/disco-config/network-policies/` - List network policies
- `GET /api/plugins/disco-config/device-policies/` - List device policies

## Contributing

Contributions are welcome! Please see [CONTRIBUTING.md](CONTRIBUTING.md) for details.

### Development Setup

```bash
# Clone repository
git clone https://github.com/yourusername/netbox-disco-config.git
cd netbox-disco-config

# Create virtual environment
python -m venv venv
source venv/bin/activate

# Install in development mode
pip install -e .
pip install -r requirements-dev.txt

# Run tests
python manage.py test netbox_disco_config
```

## Changelog

See [CHANGELOG.md](CHANGELOG.md) for version history.

## License

This project is licensed under the MIT License - see the [LICENSE](LICENSE) file for details.

## Support

- **Documentation**: [GitHub Wiki](https://github.com/yourusername/netbox-disco-config/wiki)
- **Issues**: [GitHub Issues](https://github.com/yourusername/netbox-disco-config/issues)
- **Discussions**: [GitHub Discussions](https://github.com/yourusername/netbox-disco-config/discussions)

## Credits

Developed for the NetBox community.

Special thanks to:
- NetBox Labs for the ORB agent
- NetBox community for the amazing platform
- All contributors

---

Made with ❤️ for network automation
```

---

## setup.py

```python
from setuptools import setup, find_packages

with open("README.md", "r", encoding="utf-8") as fh:
    long_description = fh.read()

setup(
    name='netbox-disco-config',
    version='1.0.0',
    description='NetBox plugin for building network discovery configurations with multi-credential and Vault support',
    long_description=long_description,
    long_description_content_type="text/markdown",
    author='Your Name',
    author_email='your.email@example.com',
    url='https://github.com/yourusername/netbox-disco-config',
    project_urls={
        'Bug Tracker': 'https://github.com/yourusername/netbox-disco-config/issues',
        'Documentation': 'https://github.com/yourusername/netbox-disco-config/wiki',
        'Source Code': 'https://github.com/yourusername/netbox-disco-config',
    },
    license='MIT',
    install_requires=[
        'netbox>=4.0.0',
    ],
    packages=find_packages(),
    include_package_data=True,
    zip_safe=False,
    keywords='netbox netbox-plugin discovery automation network vault',
    classifiers=[
        'Development Status :: 4 - Beta',
        'Framework :: Django',
        'Framework :: Django :: 4.2',
        'Intended Audience :: Developers',
        'Intended Audience :: System Administrators',
        'License :: OSI Approved :: MIT License',
        'Operating System :: OS Independent',
        'Programming Language :: Python',
        'Programming Language :: Python :: 3',
        'Programming Language :: Python :: 3.10',
        'Programming Language :: Python :: 3.11',
        'Programming Language :: Python :: 3.12',
        'Topic :: System :: Networking',
        'Topic :: System :: Systems Administration',
    ],
    python_requires='>=3.10',
)
```

---

## QUICK_START.md

```markdown
# Quick Start Guide

## Installation (5 minutes)

### Step 1: Install the Plugin

```bash
# From PyPI (when published)
pip install netbox-disco-config

# From GitHub
pip install git+https://github.com/yourusername/netbox-disco-config.git

# Local development
git clone https://github.com/yourusername/netbox-disco-config.git
cd netbox-disco-config
pip install -e .
```

### Step 2: Configure NetBox

Edit `/opt/netbox/netbox/netbox/configuration.py`:

```python
PLUGINS = [
    'netbox_disco_config',
]

PLUGINS_CONFIG = {
    'netbox_disco_config': {
        'enable_vault_integration': True,
    }
}
```

### Step 3: Run Migrations

```bash
cd /opt/netbox/netbox
python manage.py migrate netbox_disco_config
python manage.py collectstatic --no-input
```

### Step 4: Restart NetBox

```bash
sudo systemctl restart netbox netbox-rq
```

## First Configuration (10 minutes)

### 1. Create Configuration

Navigate to: **Plugins** → **Disco Configurations** → **Add**

```
Name: production
Agent Name: prod_agent_01
Diode Target: https://netbox.example.com/diode
Client ID: your_client_id
Client Secret: your_client_secret
Credential Mode: Environment Variables
```

### 2. Add Credential Set

Click on your configuration, go to **Builder** → **Credentials** tab:

```
Name: production_core
Security Zone: Production
Username: netops_prod
Password: ••••••••
Tags: production, core
```

### 3. Add Network Discovery Policy

**Network Discovery** tab:

```
Name: headquarters
Schedule: 0 */2 * * * (Every 2 hours)
Targets:
  192.168.0.0/16
  10.0.0.0/8
```

### 4. Add Device Discovery Policy

**Device Discovery** tab:

```
Name: core_switches
Schedule: 0 */6 * * * (Every 6 hours)
Credentials: production_core
Devices:
  - Driver: ios, Hostname: 10.0.1.1
  - Driver: iosxe, Hostname: 10.0.1.2
```

### 5. Generate Files

**Generate Files** tab:
- Download `.env`
- Download `agent.yaml`

### 6. Deploy

```bash
chmod 600 .env

docker run --net=host \
  -v $(pwd)/agent.yaml:/opt/orb/agent.yaml \
  --env-file .env \
  netboxlabs/orb-agent:latest run -c /opt/orb/agent.yaml
```

## Done! 🎉

Your discovery agent is now configured and running!

## Next Steps

- Add more credential sets for different zones
- Configure additional discovery policies
- Set up HashiCorp Vault integration
- Explore the REST API
- Automate deployments
```

---

## CHANGELOG.md

```markdown
# Changelog

All notable changes to this project will be documented in this file.

The format is based on [Keep a Changelog](https://keepachangelog.com/en/1.0.0/),
and this project adheres to [Semantic Versioning](https://semver.org/spec/v2.0.0.html).

## [1.0.0] - 2024-02-09

### Added
- Initial release of NetBox Disco Config
- Multi-credential management with security zones
- HashiCorp Vault integration (Token, AppRole, Kubernetes auth)
- Network discovery policy configuration
- Device discovery policy configuration
- Automatic .env and agent.yaml file generation
- REST API with full CRUD operations
- Web UI configuration builder
- Credential set tagging system
- Per-device credential overrides
- Comprehensive documentation

### Security
- Secure credential storage options
- File permission recommendations
- Vault integration for production deployments

## [Unreleased]

### Planned
- GraphQL API support
- Configuration templates
- Bulk import/export functionality
- Integration with NetBox automation features
- Advanced credential rotation policies
- Multi-tenancy support
```

---

Continue in next message with Python files...
