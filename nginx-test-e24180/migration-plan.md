# MIGRATION FROM ANSIBLE TO ANSIBLE

## Executive Summary

This repository contains legacy Ansible roles that need modernization rather than a migration from a different configuration management technology. The primary focus is on updating the existing Ansible role to follow current best practices, including using Fully Qualified Collection Names (FQCN), modern loop syntax, proper YAML formatting, and Python 3 compatibility.

The migration complexity is **LOW** as we're dealing with a single Ansible role that already follows Ansible's structure. The estimated timeline for modernization is **1-2 days** for a single developer to update the role, test it, and document the changes.

## Module Migration Plan

This repository contains Ansible roles that need individual modernization planning:

### MODULE INVENTORY

- **nginx**:
    - Description: Nginx web server installation and configuration with support for multiple sites, custom HTTP parameters, and platform-specific configurations
    - Path: roles/nginx
    - Technology: Ansible (legacy syntax)
    - Key Features: Multi-platform support (RedHat/Debian), multiple site configurations, custom HTTP parameters, SELinux compatibility

### Infrastructure Files

- `README.md`: Basic repository description, should be updated with modernization details
- `nginx-example-test-fd389c/modules/nginx/migration-plan-nginx.md`: Existing detailed migration plan for the nginx role
- `nginx-example-test-fd389c/generated-project-metadata.json`: Metadata about the project modules

### Target Details

- **Operating System**: Both RedHat and Debian family Linux distributions (based on conditional checks in tasks/main.yml)
- **Virtual Machine Technology**: Not specified, role is VM-agnostic
- **Cloud Platform**: Not specified, role is cloud-agnostic

## Migration Approach

### Key Dependencies to Address

- **libselinux-python**: Replace with python3-selinux for Python 3 compatibility
- **nginx packages**: No change needed, but ensure compatibility with target OS versions

### Security Considerations

- **SSL/TLS Configuration**: Update templates to remove deprecated TLS protocols (TLSv1, TLSv1.1) and use only TLSv1.2 and TLSv1.3
- **File Permissions**: Add explicit file permissions (mode) to all file, template, and copy tasks
- **No hardcoded credentials**: No credentials were found in the role

### Technical Challenges

- **Python 2 to 3 Compatibility**: Update Jinja2 templates to use Python 3 compatible methods (items() instead of iteritems())
- **YAML Syntax**: Convert inline key=value parameters to proper YAML dictionary format
- **Module Naming**: Update to use Fully Qualified Collection Names (FQCN)
- **Loop Syntax**: Replace with_items with modern loop syntax
- **Boolean Values**: Replace string 'yes'/'no' with true/false
- **Fact Access**: Update to use ansible_facts dictionary

### Migration Order

1. Update task syntax in main.yml (FQCN, loops, YAML format)
2. Update templates for Python 3 compatibility
3. Add argument_specs.yml for role documentation
4. Update variable defaults and types
5. Test role on both RedHat and Debian systems

### Assumptions

1. The role is intended to work on both RedHat and Debian family systems
2. The role should support multiple site configurations
3. No external dependencies beyond the standard Ansible modules are required
4. The role should be compatible with Python 3
5. No custom modules or plugins are used

## Detailed Modernization Steps

1. **Update Task Syntax**:
   - Replace module shortnames with FQCN (e.g., `yum:` → `ansible.builtin.yum:`)
   - Convert inline parameters to YAML dictionary format
   - Replace `with_items` with `loop`
   - Quote octal file modes
   - Replace string booleans with actual booleans

2. **Update Templates**:
   - Replace `iteritems()` with `items()` for Python 3 compatibility
   - Fix spacing in Jinja2 variable references
   - Update SSL protocols in nginx.conf.j2

3. **Add Documentation**:
   - Create meta/argument_specs.yml for role documentation
   - Update README.md with modernization details

4. **Testing**:
   - Test on both RedHat and Debian systems
   - Verify nginx configuration syntax
   - Test site configurations

## Conclusion

This migration is primarily a modernization effort for an existing Ansible role. The role structure is already Ansible-compatible, so the focus is on updating syntax and ensuring compatibility with current Ansible best practices and Python 3. The detailed migration plan provided in nginx-example-test-fd389c/modules/nginx/migration-plan-nginx.md contains comprehensive guidance for modernizing this role.