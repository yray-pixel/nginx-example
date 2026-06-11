---
source-path: roles/nginx
---

# Migration Plan: nginx

**TLDR**: This role installs and configures Nginx web server on both Debian and RedHat systems. It needs modernization to use FQCN module names, proper YAML syntax, quoted octal modes, modern loop syntax, and Python 3 compatibility in templates. The role also requires updates to SSL protocols and proper spacing in Jinja2 templates.

## Service Type and Configuration

**Service Type**: Web Server

**Key Operations**:
- Installs Nginx packages on RedHat and Debian systems
- Configures SELinux compatibility on RedHat systems
- Creates directory structure for site configurations
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
   - Uses legacy module names, fact access, and inline key=value syntax

2. **Install Nginx** (`roles/nginx/tasks/main.yml`):
   - Installs Nginx packages on RedHat systems using `yum` with `with_items` loop
   - Installs Nginx packages on Debian systems using `apt` with `with_items` loop and environment variables
   - Uses legacy module names, fact access, and inline key=value syntax

3. **Configure Nginx Directory Structure** (`roles/nginx/tasks/main.yml`):
   - Creates sites-available and sites-enabled directories
   - Uses legacy module name, unquoted octal mode, and with_items loop

4. **Configure Nginx** (`roles/nginx/tasks/main.yml`):
   - Copies main Nginx configuration file from template
   - Copies default configuration files
   - Creates site-specific configurations from templates
   - Creates symbolic links to enable sites
   - Uses legacy module names, missing mode parameters, and with_items loops

5. **Start Nginx Service** (`roles/nginx/tasks/main.yml`):
   - Starts and enables the Nginx service
   - Uses legacy module name and yes/no boolean

## Modernization Mapping

| Legacy Pattern | Modern Equivalent | Files Affected | Notes |
|---|---|---|---|
| `yum:` | `ansible.builtin.yum:` | tasks/main.yml | FQCN |
| `copy:` | `ansible.builtin.copy:` | tasks/main.yml | FQCN |
| `apt:` | `ansible.builtin.apt:` | tasks/main.yml | FQCN |
| `file:` | `ansible.builtin.file:` | tasks/main.yml | FQCN |
| `template:` | `ansible.builtin.template:` | tasks/main.yml | FQCN, add `mode: '0644'` to template tasks |
| `service:` | `ansible.builtin.service:` | tasks/main.yml, handlers/main.yml | FQCN |
| `with_items: redhat_pkg` | `loop: "{{ redhat_pkg }}"` | tasks/main.yml | Loop modernization with proper variable reference |
| `with_items: ubuntu_pkg` | `loop: "{{ ubuntu_pkg }}"` | tasks/main.yml | Loop modernization with proper variable reference |
| `with_items: nginx_sites` | `loop: "{{ nginx_sites }}"` | tasks/main.yml | Loop modernization with proper variable reference |
| `mode=0755` | `mode: '0755'` | tasks/main.yml | Quoted octal mode |
| `yes` | `true` | tasks/main.yml | Boolean modernization |
| `name={{ item }}` | `name: "{{ item }}"` | tasks/main.yml | YAML dictionary format |
| `ansible_os_family` | `ansible_facts['os_family']` | tasks/main.yml, templates/nginx.conf.j2 | Fact access |
| `ansible_processor_count` | `ansible_facts['processor_count']` | templates/nginx.conf.j2 | Fact access |
| `iteritems()` | `items()` | templates/nginx.conf.j2, templates/site.j2 | Python 3 compatibility |
| String boolean values like `'on'` | `true` | defaults/main.yml | Native type enforcement |
| `python-selinux` | `python3-selinux` | vars/main.yml (ubuntu_pkg list) | Python 3 compatibility |

## Dependencies

**External packages**:
- nginx
- libselinux-python (RedHat) or python3-selinux (Debian)

**Services managed**:
- nginx (start, restart, reload)

**Variables to preserve**:
- env: {'RUNLEVEL': 1}
- redhat_pkg: ['nginx']
- ubuntu_pkg: ['python-selinux', 'nginx'] (update to python3-selinux)

## Template Modernization

- **nginx.conf.j2**:
  - Replace `iteritems()` with `items()` for Python 3 compatibility
  - Fix missing spaces in variable references: `{{ nginx_log_dir}}` → `{{ nginx_log_dir }}`
  - Update SSL protocols to remove TLSv1 and TLSv1.1, use only TLSv1.2 and TLSv1.3
  - Replace `ansible_os_family` with `ansible_facts['os_family']`
  - Replace `ansible_processor_count` with `ansible_facts['processor_count']`

- **site.j2**:
  - Replace `iteritems()` with `items()` for Python 3 compatibility
  - Fix missing spaces in variable references
  - Update conditional syntax: `{% for k,v in item.server.iteritems() if k.find('location') != -1 %}` → `{% for k,v in item.server.items() if k.find('location') != -1 %}`

- **default.conf.j2** and **default.j2**:
  - Fix missing spaces in variable references
  - Replace any Python 2 specific code (iteritems, etc.)
  - Ensure proper Jinja2 spacing in all variable references

## Argument Specification

```yaml
# meta/argument_specs.yml
---
argument_specs:
  main:
    short_description: Install and configure Nginx web server
    description: Installs and configures Nginx web server on both Debian and RedHat systems
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
```

## Checks for the Migration

**Files to verify**:
- roles/nginx/tasks/main.yml
- roles/nginx/handlers/main.yml
- roles/nginx/defaults/main.yml
- roles/nginx/vars/main.yml
- roles/nginx/meta/main.yml
- roles/nginx/meta/argument_specs.yml
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
```bash
# Verify Nginx configuration syntax
sudo nginx -t

# Check Nginx service status
sudo systemctl status nginx

# Verify site configurations
ls -la /etc/nginx/sites-enabled/
ls -la /etc/nginx/sites-available/

# Test a sample HTTP request
curl -I http://localhost

# Check for SSL configuration (if applicable)
curl -I https://localhost
```