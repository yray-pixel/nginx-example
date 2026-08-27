# MIGRATION FROM ANSIBLE TO ANSIBLE

## Executive summary of migration scope, complexity, and timeline estimate

This repository contains a legacy Ansible role for Nginx that needs modernization. The role is already in Ansible format but requires updates to follow current Ansible best practices. The migration will involve updating syntax, module references, and ensuring compatibility with Python 3. The complexity is low to moderate, and the estimated timeline is 1-2 days for a complete modernization.

## Module Migration Plan

This repository contains Ansible roles that need individual migration planning:

### MODULE INVENTORY

- **nginx**:
    - Description: Nginx web server installation and configuration with support for multiple sites, custom HTTP parameters, and platform-specific configurations.
    - Path: roles/nginx
    - Technology: Ansible
    - Key Features: Multi-site configuration, EPEL repository setup for RedHat systems, platform-specific package installation, templated configuration files

### Infrastructure Files

- `roles/nginx/files/epel.repo`: EPEL repository configuration for RedHat systems, needs to be preserved
- `roles/nginx/templates/nginx.conf.j2`: Main Nginx configuration template, needs Python 3 compatibility updates
- `roles/nginx/templates/default.conf.j2`: Default server configuration template
- `roles/nginx/templates/default.j2`: Default site configuration template
- `roles/nginx/templates/site.j2`: Site-specific configuration template, needs Python 3 compatibility updates

### Target Details

Based on the source configuration files:

- **Operating System**: Both RedHat (EL 5, 6) and Debian (Ubuntu precise, quantal, raring, saucy) families are supported. The role includes conditional logic for both OS families.
- **Virtual Machine Technology**: Not specified in the configuration.
- **Cloud Platform**: No cloud-specific configurations found.

## Migration Approach

### Key Dependencies to Address

- **libselinux-python**: Replace with `python3-selinux` for Python 3 compatibility
- **nginx**: No change needed, but ensure the package is available in the target repositories

### Security Considerations

- **SSL/TLS Configuration**: The templates should be updated to use modern TLS protocols (TLSv1.2 and TLSv1.3) and remove older, insecure protocols
- **File Permissions**: Add explicit file permissions (`mode` parameter) to all file operations
- **No hardcoded credentials**: No credentials were found in the role

### Technical Challenges

- **Python 2 to Python 3 Transition**: Templates use Python 2 specific methods like `iteritems()` which need to be updated to `items()` for Python 3 compatibility
- **YAML Syntax Updates**: Convert inline key=value syntax to proper YAML dictionary format
- **Module Name Qualification**: Update module names to use Fully Qualified Collection Names (FQCN)
- **Boolean Value Standardization**: Replace string boolean values like "on" with actual boolean values (true/false)
- **Fact Access Modernization**: Update direct fact access to use the `ansible_facts` dictionary

### Migration Order

1. **Update module references to FQCN**: Replace all module references with their fully qualified names (e.g., `yum:` to `ansible.builtin.yum:`)
2. **Update YAML syntax**: Convert all inline key=value pairs to proper YAML dictionary format
3. **Update Python 2 specific code**: Replace `iteritems()` with `items()` in templates
4. **Update fact access**: Replace direct fact access with `ansible_facts` dictionary
5. **Add argument specification**: Create `meta/argument_specs.yml` for role parameters
6. **Update boolean values**: Replace string boolean values with actual boolean values
7. **Add file permissions**: Add explicit file permissions to all file operations
8. **Update loop syntax**: Replace `with_items` with `loop`
9. **Update SSL/TLS configuration**: Update SSL/TLS protocols in templates

### Assumptions

1. The target environment will run Python 3, requiring updates to Python-specific code in templates and package dependencies.
2. The role will continue to support both RedHat and Debian OS families.
3. The EPEL repository configuration is still required for RedHat systems.
4. The role will be used with Ansible 2.9 or later, requiring FQCN module names.
5. The existing directory structure and file organization will be maintained.
6. No additional features will be added during the migration, only modernization of existing functionality.
7. The role's default variables and their structure will remain the same, with updates only to value types where needed.
8. The target systems will have access to package repositories to install nginx and dependencies.