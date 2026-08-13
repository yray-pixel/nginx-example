# MIGRATION FROM ANSIBLE TO ANSIBLE

## Executive Summary

This repository contains an existing Ansible role for Nginx that requires modernization rather than a full migration from another technology. The role is functional but uses legacy syntax and practices that should be updated to follow current Ansible best practices. The modernization effort is estimated to be low to medium complexity and should take approximately 1-2 days to complete.

## Module Migration Plan

This repository contains Ansible roles that need individual modernization planning:

### MODULE INVENTORY

- **nginx**:
    - Description: Nginx web server installation and configuration with support for multiple sites, custom HTTP parameters, and platform-specific configurations
    - Path: roles/nginx
    - Technology: Ansible
    - Key Features: Multi-site configuration, platform detection (RedHat/Debian), EPEL repository configuration, custom HTTP parameters

### Infrastructure Files

- `roles/nginx/meta/main.yml`: Role metadata with dependencies and platform support
- `roles/nginx/defaults/main.yml`: Default variables for Nginx configuration
- `roles/nginx/vars/main.yml`: Platform-specific variables for package names and environment settings
- `roles/nginx/files/epel.repo`: EPEL repository configuration file for RedHat systems
- `roles/nginx/templates/*.j2`: Jinja2 templates for Nginx configuration files

### Target Details

- **Operating System**: Both RedHat (EL 5, 6, Fedora 16-18) and Debian (Ubuntu precise, quantal, raring, saucy) families are supported
- **Virtual Machine Technology**: Not specified, role is VM-agnostic
- **Cloud Platform**: Not specified, role is cloud-agnostic

## Migration Approach

Since this is not a true migration but rather a modernization of existing Ansible code, the approach will focus on updating syntax, improving security, and following current best practices.

### Key Dependencies to Address

- **python-selinux**: Update to python3-selinux for Python 3 compatibility
- **nginx**: No change needed, but ensure version compatibility with target systems

### Security Considerations

- Update SSL/TLS configuration in templates to remove deprecated protocols (TLSv1, TLSv1.1) and use only TLSv1.2 and TLSv1.3
- Add proper file permissions (mode) to all file and template tasks
- No credentials or secrets management was detected in the role

### Technical Challenges

- **Python 2 to 3 Compatibility**: Update template syntax from `iteritems()` to `items()` for Python 3 compatibility
- **YAML Syntax**: Convert inline key=value parameters to proper YAML dictionary format
- **Module Naming**: Update to Fully Qualified Collection Names (FQCN) for all modules
- **Loop Syntax**: Replace deprecated `with_items` with modern `loop` construct
- **Boolean Values**: Replace string booleans ('yes', 'no') with actual boolean values (true, false)
- **Fact Access**: Update to use `ansible_facts['fact_name']` instead of direct `ansible_fact_name` access

### Migration Order

1. Update module names to FQCN format
2. Convert inline parameters to YAML dictionary format
3. Update loop syntax from `with_items` to `loop`
4. Fix template syntax for Python 3 compatibility
5. Add proper file permissions to file and template tasks
6. Update SSL/TLS configuration in templates
7. Create argument specification in meta/argument_specs.yml
8. Update fact access to use ansible_facts dictionary
9. Update package dependencies for Python 3
10. Add proper documentation

### Assumptions

1. The role is intended to be used with both RedHat and Debian-based systems
2. The role supports multiple Nginx site configurations
3. The role is designed to work with both Python 2 and Python 3
4. No external dependencies beyond the standard Ansible modules are required
5. No secrets management or credential handling is needed
6. The role is intended to be used with Ansible 1.4 or higher (as specified in meta/main.yml)
7. The role assumes SELinux might be enabled on RedHat systems
8. The role assumes the EPEL repository is needed for RedHat systems