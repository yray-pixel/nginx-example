# MIGRATION FROM ANSIBLE TO ANSIBLE

## Executive Summary

This repository contains legacy Ansible roles that need modernization. The primary focus is on updating the existing Ansible roles to follow current best practices, improve maintainability, and ensure compatibility with newer versions of Ansible. The migration complexity is relatively low since we're modernizing within the same technology (Ansible), rather than migrating from a different configuration management tool.

**Timeline Estimate**: 1-2 weeks for complete modernization, including testing and validation.

## Module Migration Plan

This repository contains Ansible roles that need individual migration planning:

### MODULE INVENTORY

- **nginx**:
    - Description: Nginx web server installation and configuration for both RedHat and Debian systems
    - Path: roles/nginx
    - Technology: Ansible
    - Key Features: Multi-platform support (RedHat/Debian), custom site configurations, EPEL repository management, SELinux compatibility

### Infrastructure Files

- `roles/nginx/files/epel.repo`: EPEL repository configuration for RedHat systems
- `roles/nginx/templates/nginx.conf.j2`: Main Nginx configuration template
- `roles/nginx/templates/default.conf.j2`: Default Nginx configuration template
- `roles/nginx/templates/default.j2`: Default site configuration template
- `roles/nginx/templates/site.j2`: Custom site configuration template

### Target Details

- **Operating System**: Both RedHat (EL 5, 6) and Debian-based (Ubuntu precise, quantal, raring, saucy) systems
- **Virtual Machine Technology**: Not specified
- **Cloud Platform**: Not specified

## Migration Approach

### Key Dependencies to Address

- **libselinux-python**: Replace with python3-selinux for Python 3 compatibility
- **nginx**: Ensure compatibility with current versions
- **EPEL repository**: Update to latest version for current RHEL/CentOS versions

### Security Considerations

- **SSL/TLS Configuration**: Update SSL protocols in templates to remove TLSv1 and TLSv1.1, use only TLSv1.2 and TLSv1.3
- **File Permissions**: Add explicit file permissions (mode) to all file and template tasks
- **SELinux**: Ensure proper SELinux context for web server files and directories

### Technical Challenges

1. **Python 2 to Python 3 Migration**: 
   - Replace Python 2 specific methods like `iteritems()` with Python 3 compatible `items()` in templates
   - Update Python package dependencies (e.g., python-selinux to python3-selinux)

2. **Ansible Best Practices Modernization**:
   - Convert to Fully Qualified Collection Names (FQCN) for all modules
   - Replace deprecated loop syntax (`with_items`) with modern `loop` directive
   - Update YAML dictionary format (replace `key=value` with `key: value`)
   - Quote octal file modes (e.g., `mode=0755` to `mode: '0755'`)
   - Replace string boolean values with actual booleans (e.g., `'yes'` to `true`)
   - Update fact access patterns (e.g., `ansible_os_family` to `ansible_facts['os_family']`)

3. **Template Syntax Improvements**:
   - Fix inconsistent spacing in Jinja2 variable references
   - Ensure proper conditional syntax in templates

### Migration Order

1. **nginx role** (moderate complexity):
   - Update task syntax to modern Ansible standards
   - Modernize templates for Python 3 compatibility
   - Add argument specification (meta/argument_specs.yml)
   - Update dependencies for current OS versions
   - Improve security configurations

### Assumptions

1. The role is intended to work on both modern and legacy operating systems
2. No external role dependencies exist (based on empty dependencies list in meta/main.yml)
3. The role may need testing on newer OS versions not listed in meta/main.yml
4. The existing Nginx configuration is functional and only needs syntactic updates
5. No custom modules or plugins are used that would require special handling
6. The migration is focused on modernization rather than functional changes
7. No CI/CD pipeline integration is currently implemented
8. No secrets management or vault integration is present