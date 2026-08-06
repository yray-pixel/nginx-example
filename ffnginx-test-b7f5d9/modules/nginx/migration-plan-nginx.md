---
source-path: roles/nginx
---

# Migration Plan: nginx

**TLDR**: This role installs and configures Nginx web server on RedHat and Debian-based systems. It needs modernization for FQCN module names, loop syntax, boolean values, file permissions, Python 3 compatibility in templates, and proper YAML dictionary format instead of inline key=value syntax.

## Service Type and Configuration

**Service Type**: Web Server

**Key Operations**:
- Installs Nginx packages on RedHat and Debian-based systems
- Configures Nginx with main configuration file and site-specific configurations
- Creates directory structure for sites-available and sites-enabled
- Sets up default site configuration
- Manages multiple virtual hosts through variables
- Starts and enables the Nginx service

## File Structure

**Task Files:**
- roles/nginx/tasks/main.yml

**Handler Files:**
- roles/nginx/handlers/main.yml

**Variable Files:**
- roles/nginx/defaults/main.yml
- roles/nginx/vars/main.yml

**Meta:**
- roles/nginx/meta/main.yml

**Templates:**
- roles/nginx/templates/default.conf.j2
- roles/nginx/templates/default.j2
- roles/nginx/templates/nginx.conf.j2
- roles/nginx/templates/site.j2

**Static Files:**
- roles/nginx/files/epel.repo

## Module Explanation

The role performs operations in this order:

1. **Install and Configure Nginx** (`roles/nginx/tasks/main.yml`):
   - Installs SELinux Python module on RedHat systems
   - Copies EPEL repository configuration for RedHat systems
   - Installs Nginx packages on RedHat systems using a variable list
   - Installs Nginx packages on Debian systems using a variable list
   - Creates directory structure for site configurations
   - Configures main Nginx configuration file
   - Sets up default site configuration
   - Creates and enables site-specific configurations based on variables
   - Starts and enables the Nginx service

## Modernization Mapping

| Legacy Pattern | Modern Equivalent | Files Affected | Notes |
|---|---|---|---|
| `yum:` | `ansible.builtin.yum:` | tasks/main.yml | FQCN |
| `copy:` | `ansible.builtin.copy:` | tasks/main.yml | FQCN |
| `apt:` | `ansible.builtin.apt:` | tasks/main.yml | FQCN |
| `file:` | `ansible.builtin.file:` | tasks/main.yml | FQCN |
| `template:` | `ansible.builtin.template:` | tasks/main.yml | FQCN |
| `service:` | `ansible.builtin.service:` | tasks/main.yml, handlers/main.yml | FQCN |
| `with_items: redhat_pkg` | `loop: "{{ redhat_pkg }}"` | tasks/main.yml | Loop modernization with proper variable quoting |
| `with_items: ubuntu_pkg` | `loop: "{{ ubuntu_pkg }}"` | tasks/main.yml | Loop modernization with proper variable quoting |
| `with_items: nginx_sites` | `loop: "{{ nginx_sites }}"` | tasks/main.yml | Loop modernization with proper variable quoting |
| `mode=0755` | `mode: '0755'` | tasks/main.yml | Quoted octal mode |
| `update_cache=yes` | `update_cache: true` | tasks/main.yml | Boolean modernization |
| `enabled=yes` | `enabled: true` | tasks/main.yml | Boolean modernization |
| `ansible_os_family` | `ansible_facts['os_family']` | tasks/main.yml, templates/nginx.conf.j2 | Fact access |
| `ansible_processor_count` | `ansible_facts['processor_count']` | templates/nginx.conf.j2 | Fact access |
| `nginx_sites\|lower != 'none'` | `nginx_sites is not none` | tasks/main.yml | Boolean comparison |
| `iteritems()` | `items()` | templates/nginx.conf.j2, templates/site.j2 | Python 3 compatibility |
| Inline key=value syntax | YAML dictionary format | tasks/main.yml, handlers/main.yml | Syntax modernization |
| Missing mode parameter | Add `mode: '0644'` | All template and copy tasks | File permissions |
| `"on"` in nginx_http_params | `true` | defaults/main.yml | Boolean modernization |
| `"65"` in nginx_http_params | `65` | defaults/main.yml | Native type enforcement |
| `python-selinux` package | `python3-selinux` | vars/main.yml | Python 3 compatibility |

## Dependencies

**Collection dependencies** (for requirements.yml):
- ansible.builtin: 2.13.0

**Role dependencies**: None (from meta/main.yml)

**External packages**:
- libselinux-python (RedHat)
- python-selinux (Debian) - should be updated to python3-selinux
- nginx

**Services managed**: 
- nginx (started, enabled, restarted, reloaded)

## Template Modernization

- **nginx.conf.j2**: 
  - Replace `iteritems()` with `items()` for Python 3 compatibility
  - Update SSL protocols to remove TLSv1 and TLSv1.1, use only TLSv1.2 and TLSv1.3
  - Add stronger cipher suite configuration

- **site.j2**:
  - Replace `iteritems()` with `items()` for Python 3 compatibility
  - Replace string method `find` with `in` operator or `startswith()` for clearer code

## Argument Specification

Variables that should be in meta/argument_specs.yml:
- `nginx_max_clients`: integer, default: 512, description: "Maximum number of simultaneous client connections"
- `nginx_http_params`: dictionary, default: see defaults/main.yml, description: "HTTP parameters for nginx configuration"
- `nginx_log_dir`: string, default: "/var/log/nginx", description: "Directory for nginx log files"
- `nginx_access_log_name`: string, default: "access.log", description: "Name of the access log file"
- `nginx_error_log_name`: string, default: "error.log", description: "Name of the error log file"
- `nginx_separate_logs_per_site`: boolean, default: false, description: "Whether to create separate log files for each site"
- `nginx_sites`: list, default: see defaults/main.yml, description: "List of server blocks to configure"

## Checks for the Migration

**Files to verify**:
- roles/nginx/tasks/main.yml
- roles/nginx/handlers/main.yml
- roles/nginx/defaults/main.yml
- roles/nginx/vars/main.yml
- roles/nginx/meta/main.yml
- roles/nginx/meta/argument_specs.yml (new)
- roles/nginx/templates/nginx.conf.j2
- roles/nginx/templates/site.j2
- roles/nginx/templates/default.conf.j2
- roles/nginx/templates/default.j2
- roles/nginx/files/epel.repo

**Services to check**:
- nginx

**Templates to validate**:
- roles/nginx/templates/nginx.conf.j2
- roles/nginx/templates/site.j2
- roles/nginx/templates/default.conf.j2
- roles/nginx/templates/default.j2

## Pre-flight checks:
- Verify Nginx configuration: `nginx -t`
- Check Nginx service status: `systemctl status nginx`
- Validate site configurations: `curl -I http://localhost:8080` and `curl -I http://localhost:9090`
- Check for syntax errors in configuration files: `find /etc/nginx -type f -name "*.conf" -exec nginx -c {} -t \;`
- Verify SSL configuration (if applicable): `nmap --script ssl-enum-ciphers -p 443 localhost`