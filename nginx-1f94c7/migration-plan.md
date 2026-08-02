# MIGRATION FROM ANSIBLE TO ANSIBLE

## Executive Summary

This repository contains a legacy Ansible role for Nginx that needs modernization. The migration scope is relatively small, focusing on a single Nginx role that requires updating to current Ansible best practices. The estimated timeline for migration is 1-2 days for a single developer, including testing and validation.

## Module Migration Plan

This repository contains Ansible roles that need individual migration planning:

### MODULE INVENTORY

- **nginx**:
    - Description: Installs and configures the Nginx web server with customizable HTTP parameters and site configurations
    - Path: roles/nginx
    - Technology: Ansible (legacy syntax)
    - Key Features: Multi-platform support (Debian/RedHat), custom site configurations, SELinux compatibility

### Infrastructure Files

- `roles/nginx/tasks/main.yml`: Main task file for Nginx installation and configuration - requires modernization of module syntax
- `roles/nginx/handlers/main.yml`: Handlers for restarting and reloading Nginx - requires FQCN updates
- `roles/nginx/defaults/main.yml`: Default variables for Nginx configuration - requires boolean value standardization
- `roles/nginx/vars/main.yml`: Platform-specific variables - requires Python 3 compatibility updates
- `roles/nginx/templates/nginx.conf.j2`: Main Nginx configuration template - requires Python 3 compatibility and fact access updates
- `roles/nginx/templates/site.j2`: Site configuration template - requires Python 3 compatibility updates
- `roles/nginx/templates/default.conf.j2`: Default configuration template - requires spacing fixes in Jinja2 variables
- `roles/nginx/templates/default.j2`: Default site template - requires spacing fixes in Jinja2 variables
- `roles/nginx/files/epel.repo`: EPEL repository configuration for RedHat systems - no changes needed

### Target Details

- **Operating System**: Both Debian and RedHat family systems (based on conditional checks in tasks and templates)
- **Virtual Machine Technology**: Not specified, role is VM-platform agnostic
- **Cloud Platform**: Not specified, role is cloud-platform agnostic

## Migration Approach

### Key Dependencies to Address

- **python-selinux (Debian)**: Replace with python3-selinux for Python 3 compatibility
- **libselinux-python (RedHat)**: May need to be updated for newer RHEL versions
- **nginx package**: No changes needed, but consider adding version pinning for consistency

### Security Considerations

- **SSL Protocols**: Update SSL protocols in nginx.conf.j2 to remove TLSv1 and TLSv1.1, use only TLSv1.2 and TLSv1.3 for better security
- **File permissions**: Add explicit file permissions (mode) to all template and file tasks
- **No credentials detected**: The role does not contain any hardcoded credentials or secrets management

### Technical Challenges

- **Python 2 to 3 compatibility**: Templates use Python 2 specific methods like `iteritems()` that need to be updated to `items()`
- **Legacy module syntax**: Tasks use old-style module parameters (key=value) instead of YAML dictionary format
- **Missing FQCN**: Module names need to be updated to use Fully Qualified Collection Names
- **Unquoted octal modes**: File permissions use unquoted octal modes which are deprecated
- **Legacy loop syntax**: Tasks use `with_items` instead of the modern `loop` directive
- **Fact access**: Direct access to Ansible facts without using the `ansible_facts` dictionary

### Migration Order

1. **Update module syntax**: Convert all tasks to use YAML dictionary format and FQCN
2. **Update loop syntax**: Replace `with_items` with `loop` and proper variable references
3. **Fix template Python compatibility**: Replace `iteritems()` with `items()` in all templates
4. **Update fact access**: Replace direct fact access with `ansible_facts` dictionary
5. **Add argument specification**: Create meta/argument_specs.yml for role documentation
6. **Update dependencies**: Update Python dependencies for Python 3 compatibility
7. **Enhance security**: Update SSL protocols and add explicit file permissions

### Assumptions

- The role is intended to be used with both Debian and RedHat family systems
- The role is designed to configure multiple Nginx sites with custom configurations
- No external dependencies beyond the standard Nginx package and SELinux modules
- No integration with external services or authentication systems
- The role is not using any deprecated Ansible features beyond those mentioned above
- The target Ansible version is 2.9 or higher

## Detailed Modernization Mapping

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
| `name={{ item }}` | `name: "{{ item }}"` | tasks/main.yml | YAML dictionary format |
| `ansible_os_family` | `ansible_facts['os_family']` | tasks/main.yml, templates/nginx.conf.j2 | Fact access |
| `ansible_processor_count` | `ansible_facts['processor_count']` | templates/nginx.conf.j2 | Fact access |
| `iteritems()` | `items()` | templates/nginx.conf.j2, templates/site.j2 | Python 3 compatibility |
| String boolean values like `'on'` | `true` | defaults/main.yml | Native type enforcement |
| `python-selinux` | `python3-selinux` | vars/main.yml (ubuntu_pkg list) | Python 3 compatibility |
| `ssl_protocols TLSv1 TLSv1.1 TLSv1.2` | `ssl_protocols TLSv1.2 TLSv1.3` | templates/nginx.conf.j2 | Security enhancement |
| Missing spacing in templates | Add proper spacing | All templates | Fix `{{ nginx_log_dir}}` to `{{ nginx_log_dir }}`|