# MIGRATION FROM ANSIBLE TO ANSIBLE

## Executive Summary

This repository contains legacy Ansible roles that need modernization rather than a full migration from a different configuration management technology. The primary focus is on updating the existing Ansible role to follow current best practices, use fully qualified collection names (FQCN), and ensure compatibility with newer Ansible versions.

The repository contains a single Nginx role that needs modernization. The estimated timeline for this modernization is 1-2 days, with low complexity as the changes are primarily syntactical rather than functional.

## Module Migration Plan

This repository contains Ansible roles that need individual modernization planning:

### MODULE INVENTORY

- **nginx**:
    - Description: Nginx web server installation and configuration with support for multiple sites, custom HTTP parameters, and platform-specific configurations
    - Path: roles/nginx
    - Technology: Ansible
    - Key Features: Multi-site configuration, platform-specific package installation (RedHat/Debian), EPEL repository configuration, SELinux compatibility

### Infrastructure Files

- `roles/nginx/files/epel.repo`: EPEL repository configuration for RedHat systems, needed for Nginx installation
- `roles/nginx/templates/nginx.conf.j2`: Main Nginx configuration template with customizable HTTP parameters
- `roles/nginx/templates/default.conf.j2`: Default server configuration template
- `roles/nginx/templates/default.j2`: Default site configuration template
- `roles/nginx/templates/site.j2`: Template for custom site configurations

### Target Details

- **Operating System**: Both RedHat (EL 5, 6) and Debian (Ubuntu precise, quantal, raring, saucy) families are supported based on the meta/main.yml file
- **Virtual Machine Technology**: Not specified in the repository
- **Cloud Platform**: Not specified in the repository

## Migration Approach

### Key Dependencies to Address

- **libselinux-python**: Replace with python3-selinux for Python 3 compatibility on RedHat systems
- **python-selinux**: Replace with python3-selinux for Python 3 compatibility on Debian systems
- **nginx**: No change needed, but ensure the package is available in the target repositories

### Security Considerations

- **SSL/TLS Configuration**: Update SSL protocols in templates to remove deprecated TLSv1 and TLSv1.1, use only TLSv1.2 and TLSv1.3
- **File Permissions**: Add explicit mode parameters to all file, template, and copy tasks
- **Environment Variables**: The role sets RUNLEVEL=1 environment variable for package installation on Debian systems, which should be reviewed for necessity and security implications

### Technical Challenges

- **Python 3 Compatibility**: Templates use Python 2 specific methods like `iteritems()` which need to be updated to `items()` for Python 3 compatibility
- **YAML Syntax**: The role uses old inline key=value syntax which should be updated to proper YAML dictionary format
- **Module Naming**: Legacy module names need to be updated to use Fully Qualified Collection Names (FQCN)
- **Loop Syntax**: Old `with_items` loops need to be updated to use the modern `loop` directive with proper variable references
- **Boolean Values**: String boolean values like 'on' should be replaced with native boolean types
- **Fact Access**: Direct access to Ansible facts should be updated to use the `ansible_facts` dictionary

### Migration Order

1. **Update Module Names**: Replace all module references with their FQCN equivalents
2. **Modernize YAML Syntax**: Convert inline key=value syntax to proper YAML dictionary format
3. **Update Loop Syntax**: Replace `with_items` with `loop` and ensure proper variable references
4. **Fix Template Python Compatibility**: Update Python 2 specific methods in templates
5. **Add Missing Parameters**: Add mode parameters to file operations and other missing parameters
6. **Update Fact Access**: Replace direct fact access with `ansible_facts` dictionary access
7. **Create Argument Specification**: Add `meta/argument_specs.yml` for role documentation

### Assumptions

1. The role is intended to be used with both RedHat and Debian based systems
2. The role assumes the availability of EPEL repository on RedHat systems
3. The role is designed to configure multiple Nginx sites with custom configurations
4. The role assumes SELinux is enabled on RedHat systems
5. The role does not handle SSL certificate management directly
6. The role does not include advanced features like HTTP/2, rate limiting, or load balancing
7. The role assumes a specific directory structure (/etc/nginx/sites-available and /etc/nginx/sites-enabled) which may not be default on all distributions
8. The role does not handle firewall configuration for HTTP/HTTPS ports
9. The role does not include monitoring or logging configuration beyond basic log file setup