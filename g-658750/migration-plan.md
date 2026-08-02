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

- `roles/nginx/files/epel.repo`: EPEL repository configuration for RHEL/CentOS 6 systems, needs updating for newer OS versions
- `roles/nginx/templates/nginx.conf.j2`: Main Nginx configuration template with OS-specific user settings and dynamic worker processes
- `roles/nginx/templates/default.conf.j2`: Default server configuration template
- `roles/nginx/templates/default.j2`: Default site configuration template
- `roles/nginx/templates/site.j2`: Template for additional site configurations

### Target Details

Based on the source repository analysis:

- **Operating System**: Both RedHat/CentOS (versions 5, 6) and Debian-based systems (Ubuntu precise, quantal, raring, saucy) are supported. These are all older OS versions that need updating.
- **Virtual Machine Technology**: Not specified in the repository.
- **Cloud Platform**: No cloud-specific configurations detected.

## Migration Approach

### Key Dependencies to Address

- **EPEL Repository**: Current implementation uses a static EPEL 6 repository file. Replace with the `community.general.yum_repository` module for dynamic repository management.
- **OS-specific package management**: Currently using conditional `yum` and `apt` modules. Replace with the unified `package` module where possible.
- **SELinux Python Module**: Update the dependency installation approach for newer OS versions.

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

## Detailed Modernization Plan

### Syntax Modernization

| Legacy Pattern | Modern Equivalent | Files Affected | Notes |
|---|---|---|---|
| `yum:` | `ansible.builtin.yum:` | tasks/main.yml | FQCN |
| `copy:` | `ansible.builtin.copy:` | tasks/main.yml | FQCN |
| `apt:` | `ansible.builtin.apt:` | tasks/main.yml | FQCN |
| `file:` | `ansible.builtin.file:` | tasks/main.yml | FQCN |
| `template:` | `ansible.builtin.template:` | tasks/main.yml | FQCN, add `mode: '0644'` to template tasks |
| `service:` | `ansible.builtin.service:` | tasks/main.yml, handlers/main.yml | FQCN |
| `with_items: redhat_pkg` | `loop: "{{ redhat_pkg }}"` | tasks/main.yml | Loop modernization with proper variable reference |
| `with_items: ubuntu_pkg` | `loop: "{{ ubuntu_pkg }}"` | tasks/main.yml | Loop modernization with proper variable reference |
| `with_items: nginx_sites` | `loop: "{{ nginx_sites }}"` | tasks/main.yml | Loop modernization with proper variable reference |
| `mode=0755` | `mode: '0755'` | tasks/main.yml | Quoted octal mode |
| `yes` | `true` | tasks/main.yml | Boolean modernization |
| `name={{ item }}` | `name: "{{ item }}"` | tasks/main.yml | YAML dictionary format |
| `ansible_os_family` | `ansible_facts['os_family']` | tasks/main.yml, templates/nginx.conf.j2 | Fact access |
| `ansible_processor_count` | `ansible_facts['processor_count']` | templates/nginx.conf.j2 | Fact access |
| `iteritems()` | `items()` | templates/nginx.conf.j2, templates/site.j2 | Python 3 compatibility |
| String boolean values like `'on'` | `true` | defaults/main.yml | Native type enforcement |
| `python-selinux` | `python3-selinux` | vars/main.yml (ubuntu_pkg list) | Python 3 compatibility |

### Template Modernization

- **nginx.conf.j2**:
  - Replace `iteritems()` with `items()` for Python 3 compatibility
  - Fix missing spaces in variable references: `{{ nginx_log_dir}}` → `{{ nginx_log_dir }}`
  - Update SSL protocols to remove TLSv1 and TLSv1.1, use only TLSv1.2 and TLSv1.3
  - Replace `ansible_os_family` with `ansible_facts['os_family']`
  - Replace `ansible_processor_count` with `ansible_facts['processor_count']`

- **site.j2**:
  - Replace `iteritems()` with `items()` for Python 3 compatibility
  - Fix missing spaces in variable references
  - Update conditional syntax: `{% for k,v in item.server.iteritems() if k.find('location') != -1 %}` → `{% for k,v in item.server.items() if k.find('location') != -1 %}`

- **default.conf.j2** and **default.j2**:
  - Fix missing spaces in variable references
  - Replace any Python 2 specific code (iteritems, etc.)
  - Ensure proper Jinja2 spacing in all variable references

### Security Enhancements

1. Update SSL/TLS configuration:
   - Remove TLSv1 and TLSv1.1 protocols
   - Add TLSv1.3 support
   - Update cipher suites to modern recommendations
   - Add HTTPS configuration examples

2. EPEL Repository security:
   - Enable GPG checking for EPEL repository
   - Update to latest EPEL repository configuration
   - Use `community.general.yum_repository` module instead of static file

3. Add optional security headers:
   - X-Content-Type-Options
   - X-Frame-Options
   - Content-Security-Policy
   - Strict-Transport-Security

### OS Version Updates

1. Update supported OS versions:
   - RedHat/CentOS: Add support for versions 7, 8, 9
   - Ubuntu: Add support for newer LTS versions (18.04, 20.04, 22.04)
   - Remove EOL versions (EL5/6, Ubuntu precise/quantal/raring/saucy)

2. Update package dependencies:
   - Replace `python-selinux` with `python3-selinux` for newer OS versions
   - Add conditional logic for different package names across OS versions

### Role Structure Improvements

1. Add argument specification:
   - Create `meta/argument_specs.yml` with proper documentation
   - Document all variables with types, defaults, and descriptions

2. Improve variable organization:
   - Ensure consistent `nginx_` prefix for all role variables
   - Move OS-specific variables to separate files (vars/redhat.yml, vars/debian.yml)
   - Add proper documentation for all variables

3. Task organization:
   - Split main.yml into logical task files (install.yml, configure.yml, service.yml)
   - Add proper tags for selective execution
   - Improve task naming for clarity

### Testing and Validation

1. Add molecule tests:
   - Create molecule scenarios for different OS versions
   - Add idempotence tests
   - Add verification tests for nginx configuration and service

2. Add pre-flight checks:
   - Verify nginx configuration syntax
   - Test HTTP and HTTPS connectivity
   - Validate site configurations

3. Add CI/CD integration:
   - GitHub Actions workflow for testing
   - Ansible Lint configuration
   - Documentation generation