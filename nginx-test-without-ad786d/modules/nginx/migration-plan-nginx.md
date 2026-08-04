---
source-path: roles/nginx
---

# Migration Plan: nginx

**TLDR**: This role installs and configures Nginx web server on both Debian and RedHat-based systems. It needs modernization to use FQCN module names, proper YAML dictionary syntax, quoted octal modes, boolean values, loop syntax, and Python 3 compatible template code. The role also requires updates to its meta information and platform support.

## Service Type and Configuration

**Service Type**: Web Server

**Key Operations**:
- Installs Nginx packages on RedHat and Debian-based systems
- Configures SELinux support on RedHat systems
- Creates directory structure for Nginx site configurations
- Configures main Nginx configuration file
- Sets up default site configuration
- Creates and enables custom site configurations
- Manages the Nginx service (start, restart, reload)

## File Structure

**Task Files:**
roles/nginx/tasks/main.yml

**Handler Files:**
roles/nginx/handlers/main.yml

**Variable Files:**
roles/nginx/defaults/main.yml
roles/nginx/vars/main.yml

**Meta:**
roles/nginx/meta/main.yml

**Templates:**
roles/nginx/templates/default.conf.j2
roles/nginx/templates/default.j2
roles/nginx/templates/nginx.conf.j2
roles/nginx/templates/site.j2

**Static Files:**
roles/nginx/files/epel.repo

## Module Explanation

The role performs operations in this order:

1. **Install and Configure Dependencies** (`roles/nginx/tasks/main.yml`):
   - Installs SELinux Python module on RedHat systems
   - Copies EPEL repository configuration for RedHat systems
   - Legacy patterns: short module names, legacy fact access, inline key=value syntax
   - Modern equivalent: FQCN module names, structured YAML syntax, proper fact access

2. **Install Nginx** (`roles/nginx/tasks/main.yml`):
   - Installs Nginx packages on RedHat systems using `yum` module with `with_items` loop
   - Installs Nginx packages on Debian systems using `apt` module with `with_items` loop and environment variables
   - Legacy patterns: short module names, `with_items` loops, bare variables, `yes` boolean
   - Modern equivalent: FQCN module names, `loop` directive, proper variable references, `true` boolean

3. **Configure Nginx Directory Structure** (`roles/nginx/tasks/main.yml`):
   - Creates sites-available and sites-enabled directories
   - Legacy patterns: short module name, unquoted octal mode, `with_items` loop
   - Modern equivalent: FQCN module name, quoted octal mode, `loop` directive

4. **Configure Nginx** (`roles/nginx/tasks/main.yml`):
   - Copies main Nginx configuration file using templates
   - Copies default configuration files
   - Legacy patterns: short module names, missing mode parameters
   - Modern equivalent: FQCN module names, add mode parameters

5. **Configure Sites** (`roles/nginx/tasks/main.yml`):
   - Creates site configurations from templates
   - Creates symbolic links to enable sites
   - Legacy patterns: short module names, `with_items` loops, bare variables
   - Modern equivalent: FQCN module names, `loop` directive, proper variable references

6. **Start Nginx Service** (`roles/nginx/tasks/main.yml`):
   - Starts and enables the Nginx service
   - Legacy patterns: short module name, `yes` boolean
   - Modern equivalent: FQCN module name, `true` boolean

## Modernization Mapping

| Legacy Pattern | Modern Equivalent | Files Affected | Notes |
|---|---|---|---|
| `yum:` | `ansible.builtin.yum:` | tasks/main.yml | FQCN |
| `copy:` | `ansible.builtin.copy:` | tasks/main.yml | FQCN |
| `apt:` | `ansible.builtin.apt:` | tasks/main.yml | FQCN |
| `file:` | `ansible.builtin.file:` | tasks/main.yml | FQCN |
| `template:` | `ansible.builtin.template:` | tasks/main.yml | FQCN |
| `service:` | `ansible.builtin.service:` | tasks/main.yml, handlers/main.yml | FQCN |
| `with_items: redhat_pkg` | `loop: "{{ redhat_pkg }}"` | tasks/main.yml | Loop modernization with proper variable reference |
| `with_items: ubuntu_pkg` | `loop: "{{ ubuntu_pkg }}"` | tasks/main.yml | Loop modernization with proper variable reference |
| `with_items: nginx_sites` | `loop: "{{ nginx_sites }}"` | tasks/main.yml | Loop modernization with proper variable reference |
| `mode=0755` | `mode: '0755'` | tasks/main.yml | Quoted octal mode |
| `enabled=yes` | `enabled: true` | tasks/main.yml | Boolean modernization |
| `update_cache=yes` | `update_cache: true` | tasks/main.yml | Boolean modernization |
| `ansible_os_family` | `ansible_facts['os_family']` | tasks/main.yml, templates/nginx.conf.j2 | Fact access |
| `ansible_processor_count` | `ansible_facts['processor_count']` | templates/nginx.conf.j2 | Fact access |
| `iteritems()` | `items()` | templates/nginx.conf.j2, templates/site.j2 | Python 3 compatibility |
| Inline key=value syntax | YAML dictionary format | tasks/main.yml, handlers/main.yml | Syntax modernization |
| `"on"` in nginx_http_params | `true` | defaults/main.yml | Boolean modernization |
| `"65"` in nginx_http_params | `65` | defaults/main.yml | Native type enforcement |
| Missing mode parameters | Add `mode: '0644'` | template tasks in tasks/main.yml | File permission best practice |
| SSL protocols | Update to `ssl_protocols TLSv1.2 TLSv1.3;` | templates/nginx.conf.j2 | Security best practice |

## Dependencies

**Collection dependencies** (for requirements.yml):
- ansible.builtin: latest

**Role dependencies**: None (from meta/main.yml)

**External packages**:
- RedHat: libselinux-python, nginx
- Debian: python-selinux, nginx

**Services managed**: 
- nginx (started, restarted, reloaded)

## Template Modernization

- **nginx.conf.j2**: 
  - Replace `ansible_os_family` with `ansible_facts['os_family']`
  - Replace `ansible_processor_count` with `ansible_facts['processor_count']`
  - Replace `iteritems()` with `items()` for Python 3 compatibility
  - Update SSL protocols to remove TLSv1 and TLSv1.1 (security best practice)

- **site.j2**:
  - Replace `iteritems()` with `items()` for Python 3 compatibility
  - Replace string method `find` with more readable alternatives like `in` operator or `startswith`/`endswith`

## Argument Specification

```yaml
argument_specs:
  main:
    short_description: "Installs and configures Nginx web server"
    description: "Installs and configures Nginx web server with support for multiple sites"
    author: "Benno Joy (modernized)"
    options:
      nginx_max_clients:
        type: "int"
        default: 512
        description: "Maximum number of simultaneous client connections"
      nginx_http_params:
        type: "dict"
        default:
          sendfile: true
          tcp_nopush: true
          tcp_nodelay: true
          keepalive_timeout: 65
        description: "HTTP parameters for Nginx configuration"
      nginx_log_dir:
        type: "str"
        default: "/var/log/nginx"
        description: "Directory for Nginx log files"
      nginx_access_log_name:
        type: "str"
        default: "access.log"
        description: "Name of the access log file"
      nginx_error_log_name:
        type: "str"
        default: "error.log"
        description: "Name of the error log file"
      nginx_separate_logs_per_site:
        type: "bool"
        default: false
        description: "Whether to create separate log files for each site"
      nginx_sites:
        type: "list"
        elements: "dict"
        default: []
        description: "List of site configurations to create"
```

## Checks for the Migration

**Files to verify**:
- roles/nginx/tasks/main.yml
- roles/nginx/handlers/main.yml
- roles/nginx/defaults/main.yml
- roles/nginx/vars/main.yml
- roles/nginx/meta/main.yml
- roles/nginx/templates/default.conf.j2
- roles/nginx/templates/default.j2
- roles/nginx/templates/nginx.conf.j2
- roles/nginx/templates/site.j2
- roles/nginx/files/epel.repo
- roles/nginx/meta/argument_specs.yml (new file)

**Services to check**:
- nginx

**Templates to validate**:
- roles/nginx/templates/nginx.conf.j2
- roles/nginx/templates/site.j2

## Pre-flight checks:
```bash
# Verify Nginx installation
systemctl status nginx

# Verify Nginx configuration syntax
nginx -t

# Check if sites are properly configured
ls -la /etc/nginx/sites-enabled/
ls -la /etc/nginx/sites-available/

# Verify Nginx is listening on configured ports
ss -tulpn | grep nginx

# Check for any SELinux issues (on RedHat systems)
ausearch -m avc -ts recent | grep nginx
```