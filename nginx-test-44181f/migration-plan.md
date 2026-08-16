# MIGRATION FROM ANSIBLE TO ANSIBLE

## Executive Summary

This repository contains legacy Ansible roles that need modernization. The primary focus is on updating the existing nginx role to follow current Ansible best practices, addressing deprecated syntax, and ensuring compatibility with newer Ansible versions. The migration complexity is relatively low as we're dealing with a single role that already follows Ansible structure, but requires updates to syntax, module references, and security configurations.

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
- `roles/nginx/meta/main.yml`: Role metadata with outdated platform information
- `roles/nginx/tasks/main.yml`: Main tasks file with deprecated syntax and module references
- `roles/nginx/defaults/main.yml`: Default variables for the role
- `roles/nginx/vars/main.yml`: OS-specific variables

### Target Details

Based on the source repository analysis:

- **Operating System**: Both RedHat/CentOS (versions 5, 6) and Debian-based systems (Ubuntu precise, quantal, raring, saucy) are supported. These are all older OS versions that need updating to current versions.
- **Virtual Machine Technology**: Not specified in the repository.
- **Cloud Platform**: No cloud-specific configurations detected.

## Migration Approach

### Key Dependencies to Address

- **EPEL Repository**: Current implementation uses a static EPEL repository file. Replace with the `community.general.yum_repository` module for dynamic repository management.
- **OS-specific package management**: Currently using conditional `yum` and `apt` modules. Replace with the unified `ansible.builtin.package` module where possible.
- **SELinux Python Module**: Update the dependency from `libselinux-python` to `python3-selinux` for newer OS versions.
- **Nginx Package**: No version constraints are specified, should consider adding version pinning for consistency.

### Security Considerations

- **TLS Configuration**: The role likely configures older TLS protocols. Update to use only TLSv1.2 and TLSv1.3, removing older insecure protocols.
- **EPEL Repository**: Ensure GPG checking is enabled for repository security.
- **HTTPS Configuration**: Add HTTPS configuration examples and best practices.
- **File Permissions**: Add explicit file permissions to all file operations.
- **Vault/secrets management**: No credentials or secrets detected in the role.

### Technical Challenges

- **Deprecated Syntax**: The role uses older Ansible syntax like:
  - `with_items` instead of `loop`
  - Key=value parameters instead of YAML dictionary format
  - `.iteritems()` in Jinja2 templates (Python 2 specific) instead of `.items()`
  - Unquoted octal file modes
  - Non-FQCN module references

- **OS Version Support**: The role targets very old OS versions (EL5/6, Ubuntu precise/quantal) that are EOL. Need to update for current OS versions.
- **Variable Naming**: Some variables may not follow current Ansible best practices for namespacing.
- **Python 3 Compatibility**: Ensure all templates and tasks are compatible with Python 3.
- **Jinja2 Template Spacing**: Fix inconsistent spacing in Jinja2 templates.

### Migration Order

1. Update module references to use Fully Qualified Collection Names (FQCN)
2. Replace deprecated syntax with current Ansible practices
3. Update OS version support in metadata and package management
4. Enhance security configurations in templates
5. Add argument specifications for better documentation
6. Improve template handling and site configuration
7. Add proper file permissions to all file operations

### Assumptions

- The role is intended to be used with both RedHat and Debian-based systems
- The role assumes SELinux might be in use on RedHat systems
- The role is designed to support multiple nginx sites with custom configurations
- No SSL/TLS certificate management is included in the current role
- The role assumes a specific directory structure for nginx configurations (/etc/nginx/sites-available and /etc/nginx/sites-enabled)
- No integration with external services or monitoring is included