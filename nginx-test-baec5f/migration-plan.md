# MIGRATION FROM ANSIBLE TO ANSIBLE

## Executive Summary

This repository already contains Ansible roles that need to be modernized. The migration will involve updating the existing Ansible role to follow current best practices, addressing deprecated syntax, and ensuring compatibility with newer Ansible versions. The repository contains a single nginx role that appears to be an older Ansible role (compatible with Ansible 1.4+) that needs to be updated to current Ansible standards.

**Timeline Estimate**: 1-2 days for the single role migration

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

- **Operating System**: Both RedHat/CentOS and Debian-based systems are supported, but targeting older OS versions that need updating.
- **Virtual Machine Technology**: Not specified in the repository.
- **Cloud Platform**: No cloud-specific configurations detected.

## Migration Approach

### Key Dependencies to Address

- **EPEL Repository**: Current implementation uses a static EPEL repository file. Replace with the `community.general.yum_repository` module for dynamic repository management.
- **OS-specific package management**: Currently using conditional `yum` and `apt` modules. Replace with the unified `package` module where possible.
- **SELinux Python Module**: Update the dependency installation approach for newer OS versions.

### Security Considerations

- **TLS Configuration**: The role may configure older TLS protocols that should be updated to include TLS 1.3 and remove older versions for security.
- **EPEL Repository**: Repository configuration should be updated to ensure GPG checking is enabled.
- **HTTPS Configuration**: The templates should be updated to include HTTPS configuration examples.
- **Vault/secrets management**: No credentials or secrets detected in the role.

### Technical Challenges

- **Deprecated Syntax**: The role uses older Ansible syntax like `with_items` instead of `loop` and `.iteritems()` in Jinja2 templates.
- **OS Version Support**: The role may target older OS versions that are EOL. Need to update for current OS versions.
- **Variable Naming**: Some variables don't follow current Ansible best practices for namespacing.
- **Task Naming**: Some task names could be improved for clarity and consistency.
- **Module Names**: Update to use Fully Qualified Collection Names (FQCN) for all modules.

### Migration Order

1. Update variable naming and structure for consistency
2. Replace deprecated syntax with current Ansible practices
3. Update OS version support and package management
4. Enhance security configurations
5. Improve template handling and site configuration

### Assumptions

- The role is intended to be used with both RedHat and Debian-based systems
- The role assumes SELinux might be in use on RedHat systems
- The role is designed to support multiple nginx sites with custom configurations
- No SSL/TLS certificate management is included in the current role
- The role assumes a specific directory structure for nginx configurations (/etc/nginx/sites-available and /etc/nginx/sites-enabled)
- No integration with external services or monitoring is included