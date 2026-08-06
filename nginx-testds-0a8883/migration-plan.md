# MIGRATION FROM ANSIBLE (LEGACY) TO ANSIBLE (MODERN)

## Executive Summary

This repository contains a legacy Ansible role for nginx that needs to be modernized. The role was designed for Ansible 1.4+ and targets outdated operating systems (EL5/6, Ubuntu precise/quantal/raring/saucy). The migration will involve updating the role to follow current Ansible best practices, addressing deprecated syntax, ensuring compatibility with newer Ansible versions and modern operating systems, and enhancing security configurations.

**Timeline Estimate**: 1-2 days for the single role migration

## Module Migration Plan

This repository contains a legacy Ansible role that needs individual migration planning:

### MODULE INVENTORY

- **nginx**:
    - Description: Nginx web server installation and configuration with support for multiple sites, custom HTTP parameters, and platform-specific configurations
    - Path: roles/nginx
    - Technology: Ansible (legacy)
    - Key Features: Multi-site configuration, templated nginx.conf, platform detection (RedHat/Debian), EPEL repository management

### Infrastructure Files

- `roles/nginx/files/epel.repo`: EPEL repository configuration for RHEL/CentOS 6 systems, needs updating for newer OS versions
- `roles/nginx/templates/nginx.conf.j2`: Main Nginx configuration template with OS-specific user settings and dynamic worker processes
- `roles/nginx/templates/default.conf.j2`: Default server configuration template (empty except for ansible_managed comment)
- `roles/nginx/templates/default.j2`: Default site configuration template (empty except for ansible_managed comment)
- `roles/nginx/templates/site.j2`: Template for additional site configurations

### Target Details

Based on the source repository analysis:

- **Operating System**: Both RedHat/CentOS (versions 5, 6) and Debian-based systems (Ubuntu precise, quantal, raring, saucy) are supported. These are all older OS versions that need updating to current versions.
- **Virtual Machine Technology**: Not specified in the repository.
- **Cloud Platform**: No cloud-specific configurations detected.

## Migration Approach

### Key Dependencies to Address

- **EPEL Repository**: Current implementation uses a static EPEL 6 repository file. Replace with the `community.general.yum_repository` module for dynamic repository management.
- **OS-specific package management**: Currently using conditional `yum` and `apt` modules. Replace with the unified `package` module where possible.
- **SELinux Python Module**: Update the dependency installation approach for newer OS versions (python-selinux → python3-selinux).
- **Nginx Package**: The role installs nginx packages but doesn't specify versions. Consider adding version control for better consistency.

### Security Considerations

- **TLS Configuration**: The role configures `ssl_protocols TLSv1 TLSv1.1 TLSv1.2` but is missing newer TLS 1.3 and should remove older TLS versions (TLSv1, TLSv1.1) for security.
- **EPEL Repository**: Currently has `gpgcheck=0` for the main EPEL repository, which should be enabled for security.
- **No HTTPS Configuration**: The templates don't include HTTPS configuration examples, which should be added.
- **File Permissions**: Some file operations don't specify mode parameters, which should be added for security.
- **Vault/secrets management**: No credentials or secrets detected in the role.

### Technical Challenges

- **Deprecated Syntax**: The role uses older Ansible syntax:
  - `with_items` instead of `loop`
  - Inline key=value parameters instead of YAML dictionary format
  - `.iteritems()` in Jinja2 templates instead of `.items()`
  - Non-FQCN module names
  - String boolean values like `'on'` instead of native boolean types
- **OS Version Support**: The role targets very old OS versions (EL5/6, Ubuntu precise/quantal) that are EOL. Need to update for current OS versions.
- **Variable Naming**: Some variables don't follow current Ansible best practices for namespacing.
- **Task Naming**: Some task names could be improved for clarity and consistency.
- **Handler Usage**: The role uses handlers correctly but could benefit from using the `flush_handlers` directive in certain places.
- **Missing Documentation**: The role lacks proper documentation in the form of argument specifications.

### Migration Order

1. Update variable naming and structure for consistency
2. Replace deprecated syntax with current Ansible practices:
   - Replace `with_items` with `loop`
   - Use YAML dictionary format instead of inline key=value
   - Replace non-FQCN module names with FQCN
   - Update Jinja2 template syntax
3. Update OS version support and package management:
   - Replace static EPEL repository file with dynamic repository management
   - Update OS version support in meta/main.yml
   - Replace python-selinux with python3-selinux for newer OS versions
4. Enhance security configurations:
   - Update TLS protocols
   - Enable GPG checking for repositories
   - Add HTTPS configuration examples
   - Add file permission modes to all file operations
5. Improve template handling and site configuration:
   - Fix spacing issues in Jinja2 templates
   - Add proper documentation
   - Create argument specifications

### Assumptions

- The role is intended to be used with both RedHat and Debian-based systems
- The role assumes SELinux might be in use on RedHat systems
- The role is designed to support multiple nginx sites with custom configurations
- No SSL/TLS certificate management is included in the current role
- The role assumes a specific directory structure for nginx configurations (/etc/nginx/sites-available and /etc/nginx/sites-enabled)
- No integration with external services or monitoring is included