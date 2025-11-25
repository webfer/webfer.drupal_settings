# webfer.drupal_settings

A lightweight role that replaces `opdavies.drupal_settings_files` and generates
environment-specific Drupal settings files for deployments.

## Features

- Supports multiple sites
- Compatible with Ansistrano deployment directories
- Writes settings files into:

      <drupal_root>/<site>/settings.stage.php

  instead of:

      <drupal_root>/sites/<site>/settings.php

- Accepts structured variables via `drupal_settings`

## Example Usage

```yaml
drupal_settings:
  - drupal_root: '{{ release_drupal_path }}'
    sites:
      - name: 'default' # writes to: default/settings.stage.php
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
          extra_parameters: |
            $settings['deployment_identifier'] = '{{ ansistrano_release_version }}';
            $config['config_split.config_split.dev']['status'] = FALSE;
            $config['config_split.config_split.stage']['status'] = TRUE;
            $config['config_split.config_split.live']['status'] = FALSE;
```
# webfer.drupal_settings
