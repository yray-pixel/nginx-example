# MIGRATION FROM ANSIBLE TO ANSIBLE

## Executive Summary

This repository contains legacy Ansible roles that need modernization. The primary focus is on updating the nginx role to follow current Ansible best practices. The migration complexity is low to moderate, as the core functionality is already in Ansible format but requires updates to use modern syntax, module names, and Python 3 compatibility. Estimated timeline for migration: 1-2 weeks for a single developer, including testing and documentation.

## Module Migration Plan

This repository contains Ansible roles that need individual migration planning:

### MODULE INVENTORY

- **nginx**:
    - Description: Nginx web server installation and configuration with support for multiple sites, custom configurations, and platform-specific setup (RedHat/Debian)
    - Path: roles/nginx
    - Technology: Ansible (legacy syntax)
    - Key Features: Multi-site configuration, EPEL repository setup for RedHat, templated configuration files, service management

### Infrastructure Files

- `roles/nginx/files/epel.repo`: EPEL repository configuration for RedHat systems, needed for nginx package installation
- `roles/nginx/templates/nginx.conf.j2`: Main Nginx configuration template with customizable HTTP parameters
- `roles/nginx/templates/default.conf.j2`: Default server configuration template
- `roles/nginx/templates/default.j2`: Default site configuration template
- `roles/nginx/templates/site.j2`: Template for custom site configurations

### Target Details

- **Operating System**: Both RedHat (EL 5, 6) and Debian-based (Ubuntu precise, quantal, raring, saucy) systems are supported based on the meta/main.yml file
- **Virtual Machine Technology**: Not specified in the repository
- **Cloud Platform**: Not specified in the repository

## Migration Approach

### Key Dependencies to Address

- **libselinux-python**: Replace with python3-selinux for Python 3 compatibility on RedHat systems
- **python-selinux**: Replace with python3-selinux for Python 3 compatibility on Debian systems
- **nginx**: No change needed, but ensure compatibility with modern OS versions

### Security Considerations

- **SSL/TLS Configuration**: Update SSL protocols in templates to remove deprecated TLSv1 and TLSv1.1, use only TLSv1.2 and TLSv1.3
- **File Permissions**: Add explicit file permissions (mode) to all file, template, and copy tasks
- **No credentials detected**: The role does not appear to use any credentials or secrets management

### Technical Challenges

- **Python 2 to 3 Compatibility**: Templates use Python 2 specific methods like `iteritems()` which need to be replaced with `items()` for Python 3 compatibility
- **YAML Syntax Updates**: Convert inline key=value parameters to proper YAML dictionary format
- **Module Name Updates**: Replace legacy module names with Fully Qualified Collection Names (FQCN)
- **Loop Syntax Modernization**: Replace deprecated `with_items` loops with modern `loop` construct
- **Fact Access Updates**: Update direct fact access to use the `ansible_facts` dictionary

### Migration Order

1. **Update module names to FQCN**: Replace all module references with their fully qualified collection names
2. **Update YAML syntax**: Convert inline key=value parameters to proper YAML dictionary format
3. **Update loop syntax**: Replace `with_items` with `loop` and proper variable references
4. **Update fact access**: Replace direct fact access with `ansible_facts` dictionary
5. **Update Python compatibility**: Replace Python 2 specific methods in templates
6. **Add file permissions**: Add explicit file permissions to all file operations
7. **Update SSL/TLS configurations**: Ensure modern security standards in templates
8. **Create argument specifications**: Add argument_specs.yml for role documentation
9. **Update meta information**: Update supported platforms and minimum Ansible version

### Assumptions

1. The role is intended to be used with both RedHat and Debian-based systems
2. The role is designed to support multiple site configurations
3. No external dependencies beyond the standard OS packages are required
4. No secrets management or credential handling is needed
5. The role is intended to be used with Python 3
6. The existing functionality should be preserved while updating syntax and security practices