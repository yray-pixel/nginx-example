# MIGRATION FROM ANSIBLE TO ANSIBLE

## Executive Summary

This repository contains legacy Ansible roles that need modernization rather than migration from a different technology. The primary focus is on updating the existing Ansible role to follow current best practices, including fully qualified collection names (FQCN), modern YAML syntax, proper variable references, and Python 3 compatibility.

**Scope**: 1 Ansible role (nginx)
**Complexity**: Low (modernization of existing Ansible code rather than migration from another technology)
**Timeline Estimate**: 1-2 days for modernization, testing, and documentation

## Module Migration Plan

This repository contains Ansible roles that need individual modernization planning:

### MODULE INVENTORY

- **nginx**:
    - Description: Nginx web server installation and configuration with support for multiple sites, custom HTTP parameters, and platform-specific configurations
    - Path: roles/nginx
    - Technology: Ansible
    - Key Features: Multi-platform support (RedHat/Debian), custom site configurations, EPEL repository integration, SELinux compatibility

### Infrastructure Files

- `roles/nginx/README.md`: Documentation for the nginx role, including usage examples and variable descriptions
- `roles/nginx/meta/main.yml`: Role metadata including author information, supported platforms, and dependencies
- `roles/nginx/defaults/main.yml`: Default variables for the nginx role
- `roles/nginx/vars/main.yml`: Platform-specific variables for the nginx role
- `roles/nginx/files/epel.repo`: EPEL repository configuration file for RedHat systems
- `roles/nginx/templates/nginx.conf.j2`: Main Nginx configuration template
- `roles/nginx/templates/default.conf.j2`: Default server configuration template
- `roles/nginx/templates/default.j2`: Default site configuration template
- `roles/nginx/templates/site.j2`: Custom site configuration template

### Target Details

- **Operating System**: The role supports both RedHat (EL 5, 6, Fedora 16-18) and Debian (Ubuntu precise, quantal, raring, saucy) families
- **Virtual Machine Technology**: Not specified, compatible with any virtualization platform
- **Cloud Platform**: Not specified, compatible with any cloud platform

## Migration Approach

Since this is a modernization rather than a migration from a different technology, the approach will focus on updating the existing Ansible code to follow current best practices.

### Key Dependencies to Address

- **libselinux-python**: Replace with python3-selinux for Python 3 compatibility
- **nginx**: No change needed, but ensure compatibility with newer versions
- **EPEL repository**: Ensure the EPEL repository configuration is up-to-date for current RedHat versions

### Security Considerations

- **SSL/TLS Configuration**: Update SSL/TLS protocols in templates to remove deprecated protocols (TLSv1, TLSv1.1) and use only secure protocols (TLSv1.2, TLSv1.3)
- **File Permissions**: Add explicit file permissions (mode) to all file, template, and copy tasks
- **SELinux**: Ensure SELinux compatibility is maintained for RedHat systems

### Technical Challenges

- **Python 3 Compatibility**: Update Python 2 specific code (e.g., `iteritems()` to `items()`) in Jinja2 templates
- **YAML Syntax**: Convert inline key=value syntax to proper YAML dictionary format
- **Module Names**: Update to fully qualified collection names (FQCN)
- **Loop Syntax**: Replace deprecated `with_items` with modern `loop` construct
- **Fact Access**: Update fact access to use `ansible_facts['fact_name']` instead of `ansible_fact_name`
- **Boolean Values**: Replace string boolean values with actual boolean values

### Migration Order

1. **Update YAML Syntax and Module Names**: Convert all tasks to use proper YAML dictionary format and fully qualified collection names
2. **Update Loop Syntax**: Replace deprecated `with_items` with modern `loop` construct
3. **Update Fact Access**: Update fact access to use `ansible_facts['fact_name']` syntax
4. **Update Templates**: Fix Python 3 compatibility issues and Jinja2 spacing in templates
5. **Add Argument Specification**: Create `meta/argument_specs.yml` for role documentation
6. **Update Dependencies**: Update dependencies for Python 3 compatibility
7. **Update Security Configurations**: Update SSL/TLS protocols and file permissions

### Assumptions

- The role is intended to be used with Ansible 2.9 or later
- Python 3 is the target Python version
- The role should maintain backward compatibility with existing playbooks
- No major changes to the role's functionality are required
- The role should follow current Ansible best practices