# MIGRATION FROM ANSIBLE (LEGACY) TO ANSIBLE (MODERN)

## Executive Summary

This repository contains a legacy Ansible role for Nginx that needs to be modernized to follow current Ansible best practices. The role was designed for Ansible 1.4+ and targets older operating systems (EL5/6, Ubuntu precise/quantal/raring/saucy) that are now end-of-life. The migration will involve updating syntax, improving security configurations, and ensuring compatibility with current Ansible versions and modern operating systems.

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

- `roles/nginx/files/epel.repo`: EPEL repository configuration for RHEL/CentOS 6 systems, needs updating for newer OS versions
- `roles/nginx/templates/nginx.conf.j2`: Main Nginx configuration template with OS-specific user settings and dynamic worker processes
- `roles/nginx/templates/default.conf.j2`: Default server configuration template (currently empty except for ansible_managed comment)
- `roles/nginx/templates/default.j2`: Default site configuration template (currently empty except for ansible_managed comment)
- `roles/nginx/templates/site.j2`: Template for additional site configurations with dynamic location blocks

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
- **Platform Support**: Update the meta/main.yml to include current OS versions and remove EOL versions.

### Security Considerations

- **TLS Configuration**: The role configures `ssl_protocols TLSv1 TLSv1.1 TLSv1.2` but is missing newer TLS 1.3 and should remove older TLS versions for security.
- **EPEL Repository**: Currently has `gpgcheck=0` for the main EPEL repository, which should be enabled for security.
- **No HTTPS Configuration**: The templates don't include HTTPS configuration examples, which should be added.
- **Vault/secrets management**: No credentials or secrets detected in the role.

### Technical Challenges

- **Deprecated Syntax**: The role uses older Ansible syntax like `with_items` instead of `loop` and `.iteritems()` in Jinja2 templates.
- **OS Version Support**: The role targets very old OS versions (EL5/6, Ubuntu precise/quantal) that are EOL. Need to update for current OS versions.
- **Variable Naming**: Some variables don't follow current Ansible best practices for namespacing (should use `nginx_` prefix consistently).
- **Task Naming**: Some task names could be improved for clarity and consistency.
- **Handler Usage**: The role uses handlers correctly but could benefit from using the `flush_handlers` directive in certain places.
- **YAML Syntax**: The role uses older key=value syntax instead of YAML dictionary format.
- **Module Names**: The role uses short module names instead of Fully Qualified Collection Names (FQCN).

### Migration Order

1. Update meta/main.yml to support current OS versions
2. Update variable naming and structure for consistency
3. Replace deprecated syntax with current Ansible practices:
   - Replace key=value syntax with YAML dictionary format
   - Replace short module names with FQCN
   - Replace `with_items` with `loop`
   - Quote octal modes
   - Replace string booleans with actual booleans
4. Update OS version support and package management:
   - Replace static EPEL repo file with dynamic repository management
   - Update package names for current OS versions
   - Replace OS-specific package modules with unified `package` module where possible
5. Enhance security configurations:
   - Update SSL/TLS protocols
   - Enable GPG checking for repositories
   - Add HTTPS configuration examples
6. Improve template handling:
   - Replace `iteritems()` with `items()` for Python 3 compatibility
   - Fix spacing in Jinja2 variable references
   - Add proper mode parameters to template tasks
7. Add argument specifications (meta/argument_specs.yml)
8. Update documentation

### Assumptions

- The role is intended to be used with both RedHat and Debian-based systems
- The role assumes SELinux might be in use on RedHat systems
- The role is designed to support multiple nginx sites with custom configurations
- No SSL/TLS certificate management is included in the current role
- The role assumes a specific directory structure for nginx configurations (/etc/nginx/sites-available and /etc/nginx/sites-enabled)
- No integration with external services or monitoring is included