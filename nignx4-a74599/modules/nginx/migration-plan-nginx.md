---
source-path: roles/nginx
---

# Migration Plan: nginx

**TLDR**: This role installs and configures Nginx web server on RedHat and Debian-based systems. It needs modernization to use FQCN module names, proper YAML syntax, quoted octal modes, boolean values, loop syntax, and Python 3 compatibility in templates. The role also requires updates to supported platforms and package names.

## Service Type and Configuration

**Service Type**: Web Server

**Key Operations**:
- Installs Nginx packages on RedHat and Debian-based systems
- Configures SELinux dependencies on RedHat systems
- Creates directory structure for Nginx site configurations
- Configures main Nginx configuration file
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

1. **Install and Configure Dependencies** (`roles/nginx/tasks/main.yml`):
   - Installs SELinux Python module on RedHat systems
   - Copies EPEL repository configuration for RedHat systems
   - Uses legacy module names, fact access, and inline key=value syntax

2. **Install Nginx** (`roles/nginx/tasks/main.yml`):
   - Installs Nginx packages on RedHat systems using `yum` with `with_items` loop
   - Installs Nginx packages on Debian systems using `apt` with `with_items` loop and `environment: env` directive
   - Uses legacy module names, fact access, and inline key=value syntax

3. **Configure Nginx Directory Structure** (`roles/nginx/tasks/main.yml`):
   - Creates sites-available and sites-enabled directories
   - Uses legacy module name, unquoted octal mode, and inline key=value syntax

4. **Configure Nginx** (`roles/nginx/tasks/main.yml`):
   - Copies main Nginx configuration file from template
   - Copies default configuration files
   - Creates symbolic links for default site
   - Uses legacy module names, missing mode parameters, and inline key=value syntax

5. **Configure Custom Sites** (`roles/nginx/tasks/main.yml`):
   - Creates site configurations from templates using `with_items` loop
   - Creates symbolic links for site configurations
   - Uses legacy module names, bare variables in loops, and inline key=value syntax

6. **Start Nginx Service** (`roles/nginx/tasks/main.yml`):
   - Starts and enables the Nginx service
   - Uses legacy module name, `yes` instead of `true`, and inline key=value syntax

7. **Handlers** (`roles/nginx/handlers/main.yml`):
   - Restart nginx handler
   - Reload nginx handler
   - Both need FQCN updates

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
| `yes` | `true` | tasks/main.yml | Boolean value |
| `ansible_os_family` | `ansible_facts['os_family']` | tasks/main.yml, templates/nginx.conf.j2 | Fact access |
| `ansible_processor_count` | `ansible_facts['processor_count']` | templates/nginx.conf.j2 | Fact access |
| `name=libselinux-python` | `name: libselinux-python` | tasks/main.yml | YAML dictionary format |
| `name={{ item }}` | `name: "{{ item }}"` | tasks/main.yml | YAML dictionary format |
| `iteritems()` | `items()` | templates/nginx.conf.j2, templates/site.j2 | Python 3 compatibility |
| `python-selinux` | `python3-selinux` | vars/main.yml | Python 3 package name (ubuntu_pkg list only) |
| `"on"` | `true` | defaults/main.yml | Boolean value |
| `"65"` | `65` | defaults/main.yml | Native type |
| Missing `mode` parameter | Add `mode: '0644'` | All template tasks in tasks/main.yml | File permissions |
| `nginx_sites\|lower != 'none'` | `nginx_sites is not none` | tasks/main.yml | Modern comparison |
| `nginx_separate_logs_per_site == True` | `nginx_separate_logs_per_site` | templates/site.j2 | Boolean comparison |
| `k.find('location') == -1` | `'location' not in k` | templates/site.j2 | Modern string check |
| `k.find('location') != -1` | `'location' in k` | templates/site.j2 | Modern string check |

## Dependencies

**Collection dependencies**:
- No external collections required

**Role dependencies**: None (from meta/main.yml)

**External packages**:
- RedHat: libselinux-python, nginx
- Debian: python3-selinux (updated from python-selinux), nginx

**Services managed**: nginx

## Template Modernization

- **nginx.conf.j2**:
  - Replace `ansible_os_family` with `ansible_facts['os_family']`
  - Replace `ansible_processor_count` with `ansible_facts['processor_count']`
  - Replace `nginx_http_params.iteritems()` with `nginx_http_params.items()`
  - Update SSL protocols to remove deprecated TLSv1 and TLSv1.1, use only TLSv1.2 and TLSv1.3
  - Add secure SSL cipher configuration

- **site.j2**:
  - Replace `iteritems()` with `items()` for Python 3 compatibility
  - Replace `k.find('location') == -1` with `'location' not in k` for better readability
  - Replace `k.find('location') != -1` with `'location' in k` for better readability
  - Replace `nginx_separate_logs_per_site == True` with `nginx_separate_logs_per_site`

## Argument Specification

Variables that should be in meta/argument_specs.yml:
- `nginx_max_clients`: integer, default: 512, description: "Maximum number of simultaneous client connections"
- `nginx_http_params`: dictionary, default: see defaults/main.yml, description: "HTTP parameters for nginx configuration"
- `nginx_log_dir`: string, default: "/var/log/nginx", description: "Directory for nginx log files"
- `nginx_access_log_name`: string, default: "access.log", description: "Name of the access log file"
- `nginx_error_log_name`: string, default: "error.log", description: "Name of the error log file"
- `nginx_separate_logs_per_site`: boolean, default: false, description: "Whether to create separate log files for each site"
- `nginx_sites`: list, default: see defaults/main.yml, description: "List of site configurations to create"

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

**Services to check**: nginx

**Templates to validate**:
- roles/nginx/templates/nginx.conf.j2
- roles/nginx/templates/site.j2

**Meta updates needed**:
- Update outdated platform information in meta/main.yml (currently lists EL 5/6, Fedora 16/17/18, Ubuntu precise/quantal/raring/saucy)

## Pre-flight checks:
- Verify Nginx configuration: `nginx -t`
- Check Nginx service status: `systemctl status nginx`
- Verify site configurations: `ls -la /etc/nginx/sites-enabled/`
- Test HTTP access: `curl -I http://localhost`
- Test HTTPS access (if configured): `curl -I https://localhost`
- Check for SELinux issues: `audit2allow -a`
- Verify log file permissions: `ls -la /var/log/nginx/`