# MIGRATION FROM ANSIBLE TO ANSIBLE

## Executive Summary

This repository contains an existing Ansible role for Nginx that requires modernization rather than a complete migration from another technology. The migration will focus on updating the existing Ansible role to follow current best practices, addressing deprecated syntax, and ensuring compatibility with newer Ansible versions. The repository contains a single nginx role that appears to be an older Ansible role (compatible with Ansible 1.4+) that needs to be updated to current Ansible standards.

**Timeline Estimate**: 1-2 days for the single role modernization

## Module Migration Plan

This repository contains Ansible roles that need individual migration planning:

### MODULE INVENTORY

- **nginx**:
    - Description: Nginx web server installation and configuration with support for multiple sites, custom HTTP parameters, and platform-specific configurations
    - Path: roles/nginx
    - Technology: Ansible (legacy)
    - Key Features: Multi-site configuration, templated nginx.conf, platform detection (RedHat/Debian), EPEL repository management

### Infrastructure Files

- `roles/nginx/files/epel.repo`: EPEL repository configuration for RHEL/CentOS systems, needs updating for newer OS versions
- `roles/nginx/templates/nginx.conf.j2`: Main Nginx configuration template with OS-specific user settings and dynamic worker processes
- `roles/nginx/templates/default.conf.j2`: Default server configuration template
- `roles/nginx/templates/default.j2`: Default site configuration template
- `roles/nginx/templates/site.j2`: Template for additional site configurations

### Target Details

Based on the source repository analysis:

- **Operating System**: Both RedHat/CentOS and Debian-based systems are supported according to the tasks in main.yml.
- **Virtual Machine Technology**: Not specified in the repository.
- **Cloud Platform**: No cloud-specific configurations detected.

## Migration Approach

### Key Dependencies to Address

- **EPEL Repository**: Current implementation uses a static EPEL repository file. Replace with the `community.general.yum_repository` module for dynamic repository management.
- **OS-specific package management**: Currently using conditional `yum` and `apt` modules. Replace with the unified `package` module where possible.
- **SELinux Python Module**: Update the dependency installation approach for newer OS versions (python3-selinux).

### Security Considerations

- **TLS Configuration**: Update SSL protocols to remove TLSv1 and TLSv1.1, use only TLSv1.2 and TLSv1.3 for better security.
- **EPEL Repository**: Ensure GPG checking is enabled for the EPEL repository.
- **HTTPS Configuration**: Add HTTPS configuration examples and best practices.
- **Vault/secrets management**: No credentials or secrets detected in the role.

### Technical Challenges

- **Deprecated Syntax**: The role uses older Ansible syntax like:
  - `with_items` instead of `loop`
  - Inline key=value syntax instead of YAML dictionary format
  - `.iteritems()` in Jinja2 templates (Python 2 specific) instead of `.items()`
  - Unquoted octal modes
  - Legacy module names without FQCN (Fully Qualified Collection Names)
- **Fact Access**: Uses older style fact access like `ansible_os_family` instead of `ansible_facts['os_family']`
- **Boolean Values**: Uses 'yes'/'no' strings instead of true/false
- **Template Spacing**: Missing spaces in Jinja2 variable references
- **Python 3 Compatibility**: Needs updates for Python 3 compatibility in templates

### Migration Order

1. Update variable naming and structure for consistency
2. Replace deprecated syntax with current Ansible practices:
   - Replace legacy module names with FQCN
   - Update loop syntax from `with_items` to `loop`
   - Convert inline key=value syntax to YAML dictionary format
   - Quote octal modes
   - Update boolean values from 'yes'/'no' to true/false
3. Update templates for Python 3 compatibility:
   - Replace `.iteritems()` with `.items()`
   - Fix spacing in Jinja2 variable references
4. Update OS version support and package management
5. Enhance security configurations
6. Add argument specifications in meta/argument_specs.yml

### Assumptions

- The role is intended to be used with both RedHat and Debian-based systems
- The role assumes SELinux might be in use on RedHat systems
- The role is designed to support multiple nginx sites with custom configurations
- No SSL/TLS certificate management is included in the current role
- The role assumes a specific directory structure for nginx configurations (/etc/nginx/sites-available and /etc/nginx/sites-enabled)
- No integration with external services or monitoring is included