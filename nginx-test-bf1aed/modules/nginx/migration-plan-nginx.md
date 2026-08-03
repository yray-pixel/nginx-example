---
source-path: roles/nginx
---

# Migration Plan: nginx

**TLDR**: This role installs and configures Nginx web server on RedHat and Debian-based systems. It needs modernization to use FQCN module names, proper boolean values, quoted octal modes, loop syntax updates, and Python 3 compatibility in templates. The role also requires proper file permissions and Jinja2 template updates.

## Service Type and Configuration

**Service Type**: Web Server

**Key Operations**:
- Installs Nginx packages on RedHat and Debian-based systems
- Configures Nginx with main configuration file and site-specific configurations
- Creates directory structure for sites-available and sites-enabled
- Sets up default site configuration
- Creates and enables custom site configurations based on variables
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
   - Installs SELinux Python module on RedHat systems
   - Copies EPEL repository configuration for RedHat systems
   - Installs Nginx packages on RedHat systems using a variable list
   - Installs Nginx packages on Debian systems using a variable list
   - Creates directory structure for site configurations
   - Copies main Nginx configuration file
   - Copies default configuration files
   - Creates symbolic links for default site
   - Creates and enables custom site configurations
   - Starts and enables the Nginx service

2. **Handlers** (`roles/nginx/handlers/main.yml`):
   - Provides handlers to restart and reload Nginx when configuration changes

## Modernization Mapping

| Legacy Pattern | Modern Equivalent | Files Affected | Notes |
|---|---|---|---|
| `yum:` | `ansible.builtin.yum:` | tasks/main.yml | FQCN |
| `copy:` | `ansible.builtin.copy:` | tasks/main.yml | FQCN |
| `apt:` | `ansible.builtin.apt:` | tasks/main.yml | FQCN |
| `file:` | `ansible.builtin.file:` | tasks/main.yml | FQCN |
| `template:` | `ansible.builtin.template:` | tasks/main.yml | FQCN |
| `service:` | `ansible.builtin.service:` | tasks/main.yml, handlers/main.yml | FQCN |
| `with_items: redhat_pkg` | `loop: "{{ redhat_pkg }}"` | tasks/main.yml | Loop modernization with proper variable syntax |
| `with_items: ubuntu_pkg` | `loop: "{{ ubuntu_pkg }}"` | tasks/main.yml | Loop modernization with proper variable syntax |
| `with_items: nginx_sites` | `loop: "{{ nginx_sites }}"` | tasks/main.yml | Loop modernization with proper variable syntax |
| `mode=0755` | `mode: '0755'` | tasks/main.yml | Quoted octal mode |
| `yes` | `true` | tasks/main.yml | Boolean modernization |
| `False` | `false` | defaults/main.yml | Boolean modernization for nginx_separate_logs_per_site |
| `ansible_os_family` | `ansible_facts['os_family']` | tasks/main.yml, templates/nginx.conf.j2 | Fact access |
| `ansible_processor_count` | `ansible_facts['processor_count']` | templates/nginx.conf.j2 | Fact access |
| `iteritems()` | `items()` | templates/nginx.conf.j2, templates/site.j2 | Python 3 compatibility |
| Missing `mode` parameter | Add `mode: '0644'` | All template tasks and epel.repo copy task | File permissions |
| `sendfile: "on"` | `sendfile: true` | defaults/main.yml | Boolean modernization |
| `tcp_nopush: "on"` | `tcp_nopush: true` | defaults/main.yml | Boolean modernization |
| `tcp_nodelay: "on"` | `tcp_nodelay: true` | defaults/main.yml | Boolean modernization |
| `keepalive_timeout: "65"` | `keepalive_timeout: 65` | defaults/main.yml | Native type enforcement |
| `libselinux-python` | `python3-libselinux` | vars/main.yml | Python 3 compatibility for RedHat |
| `python-selinux` | `python3-selinux` | vars/main.yml | Python 3 compatibility for Debian |
| Inline key=value syntax | YAML dictionary format | tasks/main.yml, handlers/main.yml | Syntax modernization |

## Dependencies

**Collection dependencies**: None (ansible.builtin is part of ansible-core)

**Role dependencies**: None (from meta/main.yml)

**External packages**: 
- RedHat: nginx, libselinux-python (to be updated to python3-libselinux)
- Debian: nginx, python-selinux (to be updated to python3-selinux)

**Services managed**: nginx

## Template Modernization

- **nginx.conf.j2**: 
  - Replace `iteritems()` with `items()` for Python 3 compatibility
  - Fix missing whitespace in log paths: `{{ nginx_log_dir }}/{{ nginx_access_log_name }}` and `{{ nginx_log_dir }}/{{ nginx_error_log_name }}`
  - Update SSL protocols to remove TLSv1 and TLSv1.1, use only TLSv1.2 and TLSv1.3
  - Update fact access: `ansible_processor_count` to `ansible_facts['processor_count']`
  - Update fact access: `ansible_os_family` to `ansible_facts['os_family']`

- **site.j2**:
  - Replace `iteritems()` with `items()` for Python 3 compatibility
  - Replace string method `find` with more readable `in` or `startswith`
  - Fix inconsistent spacing (standardize on spaces instead of tabs)

- **default.conf.j2** and **default.j2**:
  - Review and ensure proper variable syntax with braces: `{{ variable_name }}`

## Argument Specification

```yaml
# meta/argument_specs.yml
---
argument_specs:
  main:
    short_description: Install and configure Nginx web server
    description: Installs and configures Nginx web server with custom site configurations
    author: Benno Joy
    options:
      nginx_max_clients:
        type: int
        default: 512
        description: Maximum number of simultaneous client connections
      
      nginx_http_params:
        type: dict
        default:
          sendfile: true
          tcp_nopush: true
          tcp_nodelay: true
          keepalive_timeout: 65
        description: HTTP parameters for Nginx configuration
      
      nginx_log_dir:
        type: str
        default: /var/log/nginx
        description: Directory for Nginx log files
      
      nginx_access_log_name:
        type: str
        default: access.log
        description: Name of the access log file
      
      nginx_error_log_name:
        type: str
        default: error.log
        description: Name of the error log file
      
      nginx_separate_logs_per_site:
        type: bool
        default: false
        description: Whether to create separate log files for each site
      
      nginx_sites:
        type: list
        elements: dict
        default: []
        description: List of site configurations to create
        
      env:
        type: dict
        default: {}
        description: Environment variables for apt tasks
```

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

**Services to check**: nginx

**Templates to validate**:
- roles/nginx/templates/nginx.conf.j2
- roles/nginx/templates/site.j2
- roles/nginx/templates/default.conf.j2
- roles/nginx/templates/default.j2

## Pre-flight checks:
```bash
# Verify Nginx installation
systemctl status nginx

# Verify Nginx configuration syntax
nginx -t

# Check if sites are properly configured
ls -la /etc/nginx/sites-available/
ls -la /etc/nginx/sites-enabled/

# Verify Nginx is listening on configured ports
ss -tulpn | grep nginx

# Check log files
ls -la /var/log/nginx/
```