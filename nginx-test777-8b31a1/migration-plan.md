# MIGRATION FROM ANSIBLE TO ANSIBLE

## Executive Summary

This repository contains an existing Ansible role for Nginx that requires modernization rather than a complete migration from another technology. The role is functional but uses older Ansible syntax and practices that should be updated to current standards. The migration will focus on updating deprecated syntax, improving security configurations, enhancing template handling, and ensuring compatibility with newer Ansible versions and operating systems.

Based on the repository structure and content, this is a relatively straightforward modernization project that should take approximately 1-2 days to complete. The primary focus will be on updating the single Nginx role to follow current Ansible best practices while maintaining its core functionality.

## Module Migration Plan

This repository contains Ansible roles that need individual migration planning:

### MODULE INVENTORY

- **nginx**:
    - Description: Nginx web server installation and configuration with support for multiple sites, custom HTTP parameters, and platform-specific configurations
    - Path: roles/nginx
    - Technology: Ansible (legacy)
    - Key Features: Multi-site configuration, templated nginx.conf, platform detection (RedHat/Debian), EPEL repository management

### Infrastructure Files

- `roles/nginx/files/epel.repo`: EPEL repository configuration for RedHat systems, needs updating for newer OS versions
- `roles/nginx/templates/nginx.conf.j2`: Main Nginx configuration template with OS-specific user settings and dynamic worker processes
- `roles/nginx/templates/default.conf.j2`: Default server configuration template
- `roles/nginx/templates/default.j2`: Default site configuration template
- `roles/nginx/templates/site.j2`: Template for additional site configurations
- `roles/nginx/README.md`: Documentation for the role, needs updating to reflect modernized usage
- `roles/nginx/meta/main.yml`: Role metadata, needs updating for newer Ansible versions
- `roles/nginx/defaults/main.yml`: Default variables for the role
- `roles/nginx/vars/main.yml`: OS-specific variables for the role
- `roles/nginx/handlers/main.yml`: Handlers for restarting and reloading Nginx

### Target Details

Based on the source repository analysis:

- **Operating System**: Both RedHat/CentOS and Debian-based systems are supported. The role appears to target older OS versions that need updating for current distributions.
- **Virtual Machine Technology**: Not specified in the repository.
- **Cloud Platform**: No cloud-specific configurations detected.

## Migration Approach

### Key Dependencies to Address

- **EPEL Repository**: Current implementation uses a static EPEL repository file. Replace with the `community.general.yum_repository` module for dynamic repository management.
- **OS-specific package management**: Currently using conditional `yum` and `apt` modules. Replace with the unified `ansible.builtin.package` module where possible.
- **SELinux Python Module**: Update the dependency installation approach for newer OS versions (python3-selinux instead of libselinux-python).
- **Python 2 to Python 3**: Update Jinja2 templates to use Python 3 compatible methods (e.g., replace `iteritems()` with `items()`).

### Security Considerations

- **TLS Configuration**: Update SSL protocols to remove TLSv1 and TLSv1.1, use only TLSv1.2 and TLSv1.3 for better security.
- **EPEL Repository**: Enable GPG checking for the EPEL repository.
- **HTTPS Configuration**: Add HTTPS configuration examples and best practices.
- **File Permissions**: Ensure proper file permissions are set for configuration files.
- **Vault/secrets management**: No credentials or secrets detected in the role.

### Technical Challenges

- **Deprecated Syntax**: The role uses older Ansible syntax:
  - Replace `with_items` with `loop`
  - Update inline key=value syntax to YAML dictionary format
  - Use Fully Qualified Collection Names (FQCN) for modules
  - Quote octal file modes
  - Replace `yes/no` with `true/false` for boolean values
  
- **OS Version Support**: Update package names and configurations for current OS versions.
- **Variable Naming**: Ensure consistent variable namespacing with `nginx_` prefix.
- **Task Naming**: Improve task names for clarity and consistency.
- **Template Modernization**: Fix spacing in Jinja2 templates and update Python 2 specific code.

### Migration Order

1. Update module syntax to use FQCN and modern YAML format
2. Replace deprecated loop syntax (`with_items` → `loop`)
3. Update boolean values (`yes/no` → `true/false`)
4. Modernize templates for Python 3 compatibility
5. Update package management for current OS versions
6. Enhance security configurations
7. Add argument specifications for better documentation
8. Update README.md to reflect modernized usage

### Assumptions

- The role is intended to be used with both RedHat and Debian-based systems
- The role assumes SELinux might be in use on RedHat systems
- The role is designed to support multiple nginx sites with custom configurations
- No SSL/TLS certificate management is included in the current role
- The role assumes a specific directory structure for nginx configurations (/etc/nginx/sites-available and /etc/nginx/sites-enabled)
- No integration with external services or monitoring is included