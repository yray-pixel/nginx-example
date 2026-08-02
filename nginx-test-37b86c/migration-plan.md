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

- **Operating System**: Both RedHat/CentOS and Debian-based systems are supported based on the conditional tasks in main.yml
- **Virtual Machine Technology**: Not specified in the repository
- **Cloud Platform**: No cloud-specific configurations detected

## Migration Approach

### Key Dependencies to Address

- **EPEL Repository**: Current implementation uses a static EPEL repository file. Replace with the `community.general.yum_repository` module for dynamic repository management
- **OS-specific package management**: Currently using conditional `yum` and `apt` modules. Replace with the unified `package` module where possible
- **SELinux Python Module**: Update the dependency installation approach for newer OS versions

### Security Considerations

- **TLS Configuration**: The role may need updated SSL/TLS configurations to follow current security best practices
- **EPEL Repository**: Ensure GPG checking is enabled for repository security
- **No HTTPS Configuration**: The templates may need to be updated to include HTTPS configuration examples
- **Vault/secrets management**: No credentials or secrets detected in the role

### Technical Challenges

- **Deprecated Syntax**: The role uses older Ansible syntax like `with_items` instead of `loop` and may have other deprecated patterns
- **OS Version Support**: The role may target older OS versions that need updating
- **Variable Naming**: Some variables may not follow current Ansible best practices for namespacing
- **Task Naming**: Some task names could be improved for clarity and consistency
- **Handler Usage**: The role uses handlers but could benefit from using the `flush_handlers` directive in certain places

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

## Modernization Mapping

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