# MIGRATION FROM ANSIBLE (LEGACY) TO ANSIBLE (MODERN)

This repository already contains an Ansible role that needs to be modernized. The migration will involve updating the existing Ansible role to follow current best practices, addressing deprecated syntax, and ensuring compatibility with newer Ansible versions.

## Module Migration Plan

This repository contains Ansible roles that need individual migration planning:

### MODULE INVENTORY

- **nginx**:
    - Description: Nginx web server installation and configuration with support for multiple sites, custom HTTP parameters, and platform-specific configurations
    - Path: roles/nginx
    - Technology: Ansible (legacy)
    - Key Features: Multi-site configuration, templated nginx.conf, platform detection (RedHat/Debian), EPEL repository management

**CRITICAL PATH VERIFICATION:**
The nginx role is located at roles/nginx and contains the standard Ansible role structure with tasks, handlers, templates, files, defaults, vars, and meta directories.

### Infrastructure Files

- `roles/nginx/files/epel.repo`: EPEL repository configuration for RHEL/CentOS systems, needs updating for newer OS versions
- `roles/nginx/templates/nginx.conf.j2`: Main Nginx configuration template with OS-specific user settings and dynamic worker processes
- `roles/nginx/templates/default.conf.j2`: Default server configuration template
- `roles/nginx/templates/default.j2`: Default site configuration template
- `roles/nginx/templates/site.j2`: Template for additional site configurations

### Target Details

Based on the source repository analysis:

- **Operating System**: Both RedHat/CentOS (versions 5, 6) and Debian-based systems (Ubuntu precise, quantal, raring, saucy) are supported. These are all older OS versions that need updating to current versions.
- **Virtual Machine Technology**: Not specified in the repository.
- **Cloud Platform**: No cloud-specific configurations detected.

## Migration Approach

### Key Dependencies to Address

- **EPEL Repository**: Current implementation uses a static EPEL repository file. Replace with the `community.general.yum_repository` module for dynamic repository management.
- **OS-specific package management**: Currently using conditional `yum` and `apt` modules. Replace with the unified `package` module where possible.
- **SELinux Python Module**: Update the dependency installation approach for newer OS versions (python-selinux → python3-selinux).
- **Nginx**: The core dependency remains the same, but package names may need updates for newer OS versions.

### Security Considerations

- **TLS Configuration**: The role may configure older SSL/TLS protocols. Should be updated to use only TLSv1.2 and TLSv1.3 for security.
- **EPEL Repository**: Should verify that GPG checking is enabled for the EPEL repository.
- **Vault/secrets management**: 
  - No credentials or secrets detected in the role.
  - No SSL/TLS certificate management is included in the current role.

### Technical Challenges

- **Deprecated Syntax**: The role uses older Ansible syntax:
  - `with_items` instead of `loop`
  - Inline key=value parameters instead of YAML dictionary format
  - Unquoted octal modes
  - Boolean values as strings ('on') instead of native booleans
  - Python 2 specific code like `.iteritems()` in Jinja2 templates
  - Non-FQCN module names

- **OS Version Support**: The role targets very old OS versions (EL5/6, Ubuntu precise/quantal/raring/saucy) that are EOL. Need to update for current OS versions.

- **Variable Naming**: Some variables don't follow current Ansible best practices for namespacing.

- **Python 3 Compatibility**: The role was designed for Python 2 and needs updates for Python 3 compatibility:
  - Replace `iteritems()` with `items()` in templates
  - Update Python package names (python-selinux → python3-selinux)

### Migration Order

1. Update module references to use Fully Qualified Collection Names (FQCN)
2. Replace deprecated syntax with current Ansible practices
3. Update OS version support and package management
4. Enhance security configurations
5. Improve template handling and site configuration
6. Add argument specifications for better documentation

### Assumptions

- The role is intended to be used with both RedHat and Debian-based systems
- The role assumes SELinux might be in use on RedHat systems
- The role is designed to support multiple nginx sites with custom configurations
- No SSL/TLS certificate management is included in the current role
- The role assumes a specific directory structure for nginx configurations (/etc/nginx/sites-available and /etc/nginx/sites-enabled)
- No integration with external services or monitoring is included