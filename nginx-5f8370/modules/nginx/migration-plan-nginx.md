---
source-path: roles/nginx
---

# Migration Plan: nginx

**TLDR**: This role installs and configures Nginx web server on RedHat and Debian-based systems. It needs modernization for FQCN module names, loop syntax, boolean values, file permissions, Python 3 compatibility in templates, and proper YAML dictionary format instead of inline key=value syntax.

## Service Type and Configuration

**Service Type**: Web Server

**Key Operations**:
- Installs Nginx packages on RedHat and Debian-based systems
- Configures SELinux support on RedHat systems
- Creates site configuration directories
- Configures Nginx with main configuration file and site-specific configurations
- Sets up default site and custom sites based on variables
- Manages the Nginx service (start, restart, reload)

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
   - Installs SELinux Python module on RedHat systems using short module name `yum`
   - Copies EPEL repository configuration for RedHat systems using short module name `copy` without mode parameter
   - Installs Nginx packages on RedHat systems using `with_items` loop and short module name `yum`
   - Installs Nginx packages on Debian systems using `with_items` loop and short module name `apt` with string boolean `yes` and environment directive `environment: env`
   - Creates site configuration directories using short module name `file` with unquoted octal mode
   - Configures Nginx using templates with short module name `template` without mode parameter
   - Creates symbolic links for site configurations using short module name `file`
   - Starts the Nginx service using short module name `service` with string boolean `yes`

2. **Handlers** (`roles/nginx/handlers/main.yml`):
   - Defines handlers for restarting and reloading Nginx using short module name `service`
   - Both handlers (restart nginx and reload nginx) will be preserved with their original names

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
| `yes` | `true` | tasks/main.yml | Boolean modernization |
| `ansible_os_family` | `ansible_facts['os_family']` | tasks/main.yml, templates/nginx.conf.j2 | Fact access |
| `ansible_processor_count` | `ansible_facts['processor_count']` | templates/nginx.conf.j2 | Fact access |
| `iteritems()` | `items()` | templates/nginx.conf.j2 (line 24), templates/site.j2 (lines 8, 14) | Python 3 compatibility |
| Inline key=value syntax | YAML dictionary format | tasks/main.yml, handlers/main.yml | Syntax modernization |
| Missing mode parameter | Add `mode: '0644'` | tasks/main.yml (template and copy tasks) | File permissions |
| `"on"` in nginx_http_params | `true` | defaults/main.yml (sendfile, tcp_nopush, tcp_nodelay) | Boolean modernization |
| String integer `"65"` | Integer `65` | defaults/main.yml | Native type enforcement |
| `python-selinux` package | `python3-selinux` package | vars/main.yml, ubuntu_pkg list | Python 3 compatibility |
| SSL protocols | Remove `TLSv1` and `TLSv1.1` | templates/nginx.conf.j2 | Security enhancement |

## Dependencies

**External packages**:
- RedHat: libselinux-python, nginx
- Debian: python-selinux (should be updated to python3-selinux), nginx

**Role dependencies**: None (from meta/main.yml)

**Services managed**: nginx

## Template Modernization

- **nginx.conf.j2**: 
  - Replace `ansible_os_family` with `ansible_facts['os_family']`
  - Replace `ansible_processor_count` with `ansible_facts['processor_count']`
  - Replace `iteritems()` with `items()` on line 24 for Python 3 compatibility
  - Update SSL protocols to remove TLSv1 and TLSv1.1 for better security

- **site.j2**:
  - Replace `iteritems()` with `items()` on lines 8 and 14 for Python 3 compatibility
  - Replace string comparison method `find` with more readable `in` operator or `startswith`/`endswith`

## Argument Specification

Variables that should be in meta/argument_specs.yml:
- `nginx_max_clients`: integer, default: 512, description: "Maximum number of simultaneous client connections"
- `nginx_http_params`: dictionary, default: {'sendfile': true, 'tcp_nopush': true, 'tcp_nodelay': true, 'keepalive_timeout': 65}, description: "HTTP parameters for Nginx configuration"
- `nginx_log_dir`: string, default: "/var/log/nginx", description: "Directory for Nginx logs"
- `nginx_access_log_name`: string, default: "access.log", description: "Name of access log file"
- `nginx_error_log_name`: string, default: "error.log", description: "Name of error log file"
- `nginx_separate_logs_per_site`: boolean, default: false, description: "Whether to create separate log files for each site"
- `nginx_sites`: list, default: [site configurations], description: "List of site configurations to create"

## Checks for the Migration

**Files to verify**:
- roles/nginx/tasks/main.yml
- roles/nginx/handlers/main.yml
- roles/nginx/defaults/main.yml
- roles/nginx/vars/main.yml
- roles/nginx/meta/main.yml
- roles/nginx/meta/argument_specs.yml (new file)
- roles/nginx/templates/default.conf.j2
- roles/nginx/templates/default.j2
- roles/nginx/templates/nginx.conf.j2
- roles/nginx/templates/site.j2
- roles/nginx/files/epel.repo

**Services to check**: nginx

**Templates to validate**:
- roles/nginx/templates/nginx.conf.j2
- roles/nginx/templates/site.j2

## Pre-flight checks:
- Verify Nginx configuration: `nginx -t`
- Check Nginx service status: `systemctl status nginx`
- Verify site configurations: `ls -la /etc/nginx/sites-enabled/`
- Test HTTP response: `curl -I http://localhost`
- Check for SELinux issues on RedHat systems: `ausearch -m avc -ts recent`