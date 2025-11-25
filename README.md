# 🔧 webfer.drupal_settings

An Ansible role for generating Drupal settings.php files with structured configuration management. This role is particularly useful for automated deployments and managing multiple Drupal environments with different configurations.

## 📋 Description

This Ansible role automates the creation of Drupal `settings.php` files by using Jinja2 templates and structured YAML configuration. It's designed to work seamlessly with deployment tools like Ansistrano and supports managing multiple Drupal sites from a single configuration.

The role generates PHP settings files with database configurations, security settings, trusted hosts, and custom parameters, making it easy to maintain consistent configurations across different environments (development, staging, production).

## ✨ Features

- **Multiple Site Support**: Configure multiple Drupal sites in a single playbook
- **Ansistrano Compatible**: Works with Ansistrano deployment directory structures
- **Flexible File Placement**: Write settings files to custom locations (e.g., `default/settings.stage.php` instead of `sites/default/settings.php`)
- **Structured Configuration**: Use YAML variables for database connections, trusted hosts, hash salt, and more
- **Sensitive Data Protection**: Uses `no_log: true` to prevent sensitive information from appearing in logs
- **Custom Parameters**: Inject additional PHP code via `extra_parameters` for environment-specific settings

## 📦 Requirements

- Ansible 2.0 or higher
- Target system with appropriate permissions to write to Drupal directories

## 🚀 Installation

### From Git Repository

```bash
ansible-galaxy install git+https://github.com/webfer/webfer.drupal_settings.git
```

Or add to your `requirements.yml`:

```yaml
- src: git+https://github.com/webfer/webfer.drupal_settings.git
  name: webfer.drupal_settings
```

## 🎛️ Role Variables

### 🔑 Required Variables

- `drupal_settings`: List of Drupal configurations (default: `[]`)

### 📐 Configuration Structure

```yaml
drupal_settings:
  - drupal_root: '<path_to_drupal_root>'
    sites:
      - name: 'default' # Site directory name (default: 'default')
        filename: 'settings.php' # Settings filename (default: 'settings.php')
        settings:
          databases: {} # Database configuration
          base_url: '' # Optional base URL
          hash_salt: '' # Optional hash salt
          config_directories: {} # Optional config directories
          trusted_hosts: [] # Optional trusted host patterns
          extra_parameters: '' # Optional raw PHP code
```

## 💡 Usageage

### 🎯 Using include_role

You can include this role in your playbook tasks using `include_role`:

```yaml
- name: Generate settings.php file
  include_role:
    name: webfer.drupal_settings
  tags:
    - deploy
```

This approach is useful when you need to call the role conditionally or multiple times within a playbook.

### 📝 Basic Example

```yaml
- hosts: webservers
  roles:
    - role: webfer.drupal_settings
      drupal_settings:
        - drupal_root: '/var/www/drupal'
          sites:
            - name: 'default'
              filename: 'settings.php'
              settings:
                databases:
                  default:
                    default:
                      driver: mysql
                      host: localhost
                      port: '3306'
                      database: 'drupal_db'
                      username: 'drupal_user'
                      password: 'secure_password'
                      prefix: ''
                      collation: 'utf8mb4_general_ci'
                      isolevel: 'SET SESSION TRANSACTION ISOLATION LEVEL READ COMMITTED'
                hash_salt: 'your_random_hash_salt_here'
                trusted_hosts:
                  - '^example\.com$'
                  - '^www\.example\.com$'
```

### 🚢 Advanced Example with Ansistrano

```yaml
- hosts: webservers
  roles:
    - role: webfer.drupal_settings
      drupal_settings:
        - drupal_root: '{{ ansistrano_release_path.stdout }}'
          sites:
            - name: 'default'
              filename: 'settings.stage.php'
              settings:
                databases:
                  default:
                    default:
                      driver: mysql
                      host: localhost
                      port: '{{ vault_database_port }}'
                      database: '{{ vault_database_name }}'
                      username: '{{ vault_database_user }}'
                      password: '{{ vault_database_password }}'
                      prefix: '{{ vault_database_prefix }}'
                      collation: '{{ vault_database_collation }}'
                      isolevel: 'SET SESSION TRANSACTION ISOLATION LEVEL READ COMMITTED'
                base_url: 'https://stage.example.com'
                hash_salt: '{{ vault_hash_salt }}'
                trusted_hosts:
                  - '^stage\.example\.com$'
                extra_parameters: |
                  $settings['deployment_identifier'] = '{{ ansistrano_release_version }}';
                  $config['config_split.config_split.dev']['status'] = FALSE;
                  $config['config_split.config_split.stage']['status'] = TRUE;
                  $config['config_split.config_split.live']['status'] = FALSE;
```

### 🌐 Multiple Sites Example

```yaml
drupal_settings:
  - drupal_root: '/var/www/drupal'
    sites:
      - name: 'site1'
        filename: 'settings.php'
        settings:
          databases:
            default:
              default:
                driver: mysql
                host: localhost
                database: 'site1_db'
                username: 'site1_user'
                password: 'password1'
      - name: 'site2'
        filename: 'settings.php'
        settings:
          databases:
            default:
              default:
                driver: mysql
                host: localhost
                database: 'site2_db'
                username: 'site2_user'
                password: 'password2'
```

## ⚙️ Configuration Optionsons

### 🗄️ Database Settings

All standard Drupal database connection parameters are supported:

- `driver`: Database driver (mysql, pgsql, sqlite, etc.)
- `host`: Database hostname
- `port`: Database port
- `database`: Database name
- `username`: Database username
- `password`: Database password
- `prefix`: Table prefix
- `collation`: Database collation
- `isolevel`: Transaction isolation level

### 🔧 Optional Settings

- **base_url**: Set the base URL for the site
- **hash_salt**: Drupal's hash salt for security
- **config_directories**: Configuration sync directories (Drupal 8)
- **trusted_hosts**: Array of trusted host patterns (regex)
- **extra_parameters**: Raw PHP code injected at the end of settings file

## 📄 Generated File Structure

The role creates settings files with the following structure:

```php
<?php

// Ansible managed

$databases['default']['default'] = array(
  'driver'    => 'mysql',
  'host'      => 'localhost',
  // ... other database settings
);

$base_url = 'https://example.com';
$settings['hash_salt'] = 'your_hash_salt';
$settings['trusted_host_patterns'] = array(
  '^example\.com$',
);

// Custom parameters
$settings['deployment_identifier'] = '20231125120000';
```

## 🔒 Security Considerations

- Use Ansible Vault for sensitive data (passwords, hash salts)
- The role uses `no_log: true` to prevent sensitive data from appearing in logs
- Ensure proper file permissions (default: 0640) on generated settings files
- Use trusted_hosts patterns to prevent HTTP Host header attacks

## 📜 Licensense

MIT

## 👤 Author Information

This role was created by [webfer](https://www.linkedin.com/in/webfer/).
