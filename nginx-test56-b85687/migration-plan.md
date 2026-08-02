# MIGRATION FROM ANSIBLE TO ANSIBLE

## Executive Summary

This repository contains an existing Ansible role for Nginx that requires modernization rather than a full migration from another technology. The role is functional but uses legacy Ansible syntax and practices that should be updated to follow current Ansible best practices. The estimated timeline for modernization is 1-2 days for a single developer, as the changes are primarily syntactical rather than functional.

## Module Migration Plan

This repository contains Ansible roles that need modernization:

### MODULE INVENTORY

- **nginx**:
    - Description: Nginx web server installation and configuration with support for multiple sites, custom HTTP parameters, and platform-specific configurations
    - Path: roles/nginx
    - Technology: Ansible
    - Key Features: Multi-site configuration, EPEL repository setup for RedHat systems, platform-specific package installation, templated configuration files

### Infrastructure Files

- `roles/nginx/files/epel.repo`: EPEL repository configuration for RedHat systems - should be preserved as-is
- `roles/nginx/templates/nginx.conf.j2`: Main Nginx configuration template - requires Python 3 compatibility updates
- `roles/nginx/templates/default.conf.j2`: Default server configuration template - requires spacing fixes in Jinja2 syntax
- `roles/nginx/templates/default.j2`: Default site configuration template - requires spacing fixes in Jinja2 syntax
- `roles/nginx/templates/site.j2`: Custom site configuration template - requires Python 3 compatibility updates

### Target Details

- **Operating System**: Both RedHat (EL 5, 6, Fedora 16-18) and Debian (Ubuntu precise, quantal, raring, saucy) families are supported based on the meta/main.yml file
- **Virtual Machine Technology**: Not specified in the repository
- **Cloud Platform**: Not specified in the repository

## Migration Approach

### Key Dependencies to Address

- **python-selinux**: Update to python3-selinux for Python 3 compatibility on Debian systems
- **libselinux-python**: Verify compatibility with newer RHEL versions

### Security Considerations

- SSL/TLS configuration: Update templates to remove deprecated TLS protocols (TLSv1, TLSv1.1) and use only TLSv1.2 and TLSv1.3
- No hardcoded credentials were detected in the role
- No vault usage was detected

### Technical Challenges

- **Python 2 to Python 3 compatibility**: Templates use Python 2 specific methods like `iteritems()` which need to be updated to `items()` for Python 3 compatibility
- **Legacy Ansible syntax**: The role uses older Ansible syntax patterns that need to be updated:
  - Legacy module names without FQCN (Fully Qualified Collection Names)
  - Key=value parameter format instead of YAML dictionary format
  - Unquoted octal file modes
  - Deprecated loop syntax (`with_items` instead of `loop`)
  - Boolean values as strings (`yes`/`no` instead of `true`/`false`)
  - Direct fact access instead of using `ansible_facts` dictionary

### Migration Order

1. Update task syntax in `roles/nginx/tasks/main.yml` (low risk, high value)
2. Update handler syntax in `roles/nginx/handlers/main.yml` (low risk)
3. Update Python 2 specific code in templates (moderate complexity)
4. Add argument specification in `meta/argument_specs.yml` (moderate complexity)
5. Update SSL/TLS configuration in templates (moderate risk)

### Assumptions

1. The role is intended to be used with both Python 2 and Python 3 environments
2. The role should maintain backward compatibility with existing playbooks
3. The role should work with both older and newer versions of Ansible
4. No functional changes are required, only syntactical modernization
5. The existing directory structure should be preserved
6. The role should continue to support both RedHat and Debian family systems
7. The existing variable names and defaults should be preserved for backward compatibility