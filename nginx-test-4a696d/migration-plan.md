# MIGRATION FROM ANSIBLE TO ANSIBLE

## Executive Summary

This repository contains an existing Ansible role for Nginx that requires modernization rather than a full migration from another technology. The role is functional but uses legacy Ansible syntax and practices that should be updated to follow current Ansible best practices. The modernization effort is estimated to be low to medium complexity and could be completed in 1-2 days by an experienced Ansible developer.

## Module Migration Plan

This repository contains Ansible roles that need individual modernization planning:

### MODULE INVENTORY

- **nginx**:
    - Description: Nginx web server installation and configuration with support for multiple sites, custom HTTP parameters, and platform-specific configurations
    - Path: roles/nginx
    - Technology: Ansible
    - Key Features: Multi-site configuration, EPEL repository management, platform-specific package installation (RedHat/Debian), custom HTTP parameters

### Infrastructure Files

- `roles/nginx/files/epel.repo`: EPEL repository configuration for RedHat systems, needed for Nginx installation
- `roles/nginx/templates/nginx.conf.j2`: Main Nginx configuration template
- `roles/nginx/templates/default.conf.j2`: Default server configuration template
- `roles/nginx/templates/default.j2`: Default site configuration template
- `roles/nginx/templates/site.j2`: Template for custom site configurations

### Target Details

Based on the source configuration files:

- **Operating System**: Both RedHat (EL 5, 6) and Debian (Ubuntu precise, quantal, raring, saucy) families are supported
- **Virtual Machine Technology**: Not specified
- **Cloud Platform**: Not specified, the role appears to be designed for on-premises or generic cloud deployments

## Migration Approach

### Key Dependencies to Address

- **libselinux-python**: Update to use `python3-selinux` for Python 3 compatibility on modern systems
- **nginx**: No change needed, but ensure compatibility with newer versions

### Security Considerations

- **SSL/TLS Configuration**: The templates should be updated to remove deprecated SSL/TLS protocols (TLSv1, TLSv1.1) and use only TLSv1.2 and TLSv1.3
- **File Permissions**: Add explicit file permissions (`mode` parameter) to all file operations
- **No credentials detected**: The role does not appear to contain any hardcoded credentials or sensitive information

### Technical Challenges

- **Python 2 to Python 3 Compatibility**: Templates use Python 2 specific methods like `iteritems()` which need to be updated to `items()` for Python 3 compatibility
- **Legacy Ansible Syntax**: The role uses old-style Ansible syntax (key=value, unquoted modes) that should be updated to YAML dictionary format
- **FQCN Module Names**: Update module references to use Fully Qualified Collection Names (FQCN)
- **Loop Syntax**: Replace deprecated `with_items` with modern `loop` construct
- **Fact Access**: Update direct fact access to use the `ansible_facts` dictionary

### Migration Order

1. Update Python 2 specific code in templates to Python 3 compatible code
2. Convert legacy Ansible syntax to modern YAML dictionary format
3. Update module references to use FQCN
4. Replace deprecated loop syntax
5. Update fact access patterns
6. Add explicit file permissions
7. Update SSL/TLS configurations
8. Create argument specification file
9. Update meta information
10. Test the modernized role

### Assumptions

1. The role is intended to be used with both RedHat and Debian based systems
2. The role is designed to support multiple Nginx sites with custom configurations
3. The role is expected to work with both Python 2 and Python 3
4. No external dependencies beyond the standard Ansible modules are required
5. The role is intended to be used with Ansible 1.4 or higher (as specified in meta/main.yml)
6. The role does not require any special privileges beyond those needed to install packages and configure services

## Detailed Modernization Steps

### 1. Update Python 2 specific code in templates

- Replace `iteritems()` with `items()` in all templates
- Fix missing spaces in variable references (e.g., `{{ nginx_log_dir}}` → `{{ nginx_log_dir }}`)

### 2. Convert legacy Ansible syntax to modern YAML dictionary format

- Replace `key=value` syntax with `key: value` format
- Quote octal modes (e.g., `mode=0755` → `mode: '0755'`)
- Replace string booleans with actual booleans (e.g., `sendfile: "on"` → `sendfile: true`)

### 3. Update module references to use FQCN

- `yum:` → `ansible.builtin.yum:`
- `copy:` → `ansible.builtin.copy:`
- `apt:` → `ansible.builtin.apt:`
- `file:` → `ansible.builtin.file:`
- `template:` → `ansible.builtin.template:`
- `service:` → `ansible.builtin.service:`

### 4. Replace deprecated loop syntax

- `with_items: redhat_pkg` → `loop: "{{ redhat_pkg }}"`
- `with_items: ubuntu_pkg` → `loop: "{{ ubuntu_pkg }}"`
- `with_items: nginx_sites` → `loop: "{{ nginx_sites }}"`

### 5. Update fact access patterns

- `ansible_os_family` → `ansible_facts['os_family']`
- `ansible_processor_count` → `ansible_facts['processor_count']`

### 6. Add explicit file permissions

- Add `mode: '0644'` to all template tasks
- Add `mode: '0644'` to all copy tasks

### 7. Update SSL/TLS configurations

- Remove TLSv1 and TLSv1.1 from SSL protocols
- Use only TLSv1.2 and TLSv1.3

### 8. Create argument specification file

- Create `meta/argument_specs.yml` with proper documentation of all role variables

### 9. Update meta information

- Update `min_ansible_version` to a more recent version (e.g., 2.9)
- Replace `categories` with `galaxy_tags`
- Add `namespace` if applicable

### 10. Test the modernized role

- Verify syntax with `ansible-playbook --syntax-check`
- Run the role against a test environment
- Verify Nginx configuration and service status