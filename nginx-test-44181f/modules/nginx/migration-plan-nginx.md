---
source-path: roles/nginx
---

# Migration Plan: nginx

**TLDR**: This role installs and configures Nginx web server on RedHat and Debian-based systems. It needs modernization to use FQCN module names, proper YAML dictionary syntax, quoted octal modes, modern loop syntax, and Python 3 compatibility in templates. The role also requires updates to its supported platforms and package dependencies.

## Service Type and Configuration

**Service Type**: Web Server

**Key Operations**:
- Installs Nginx packages on RedHat and Debian-based systems
- Creates directory structure for site configurations
- Configures Nginx with main configuration file and default site
- Sets up custom site configurations based on variables
- Manages Nginx service (start, restart, reload)
- Configures SELinux support on RedHat systems

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
   - Installs Nginx packages on Debian systems using `apt` with `with_items` loop
   - Uses legacy module names, fact access, and inline key=value syntax

3. **Configure Nginx** (`roles/nginx/tasks/main.yml`):
   - Creates directory structure for site configurations
   - Copies main Nginx configuration file from template
   - Copies default configuration files
   - Creates links for default site
   - Uses legacy module names, unquoted octal modes, and missing mode parameters

4. **Configure Custom Sites** (`roles/nginx/tasks/main.yml`):
   - Creates site configurations from templates using `with_items` loop
   - Creates symbolic links to enable sites using `with_items` loop
   - Uses legacy module names, bare variables in loops, and missing mode parameters

5. **Start Service** (`roles/nginx/tasks/main.yml`):
   - Starts and enables the Nginx service
   - Uses legacy module name and `yes` instead of `true`

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
| `name=libselinux-python` | `name: libselinux-python` | tasks/main.yml | YAML dictionary syntax |
| `src=epel.repo dest=/etc/yum.repos.d/epel_ansible.repo` | `src: epel.repo`<br>`dest: /etc/yum.repos.d/epel_ansible.repo` | tasks/main.yml | YAML dictionary syntax |
| `name={{ item }} state=present` | `name: "{{ item }}"`<br>`state: present` | tasks/main.yml | YAML dictionary syntax |
| `name={{ item }} state=present update_cache=yes` | `name: "{{ item }}"`<br>`state: present`<br>`update_cache: true` | tasks/main.yml | YAML dictionary syntax and boolean modernization |
| `path=/etc/nginx/{{ item }} state=directory` | `path: "/etc/nginx/{{ item }}"`<br>`state: directory` | tasks/main.yml | YAML dictionary syntax |
| `src=nginx.conf.j2 dest=/etc/nginx/nginx.conf` | `src: nginx.conf.j2`<br>`dest: /etc/nginx/nginx.conf`<br>`mode: '0644'` | tasks/main.yml | YAML dictionary syntax and adding mode parameter |
| `path=/etc/nginx/sites-enabled/default state=link src=/etc/nginx/sites-available/default` | `path: /etc/nginx/sites-enabled/default`<br>`state: link`<br>`src: /etc/nginx/sites-available/default` | tasks/main.yml | YAML dictionary syntax |
| `name=nginx state=started enabled=yes` | `name: nginx`<br>`state: started`<br>`enabled: true` | tasks/main.yml | YAML dictionary syntax and boolean modernization |
| `name=nginx state=restarted` | `name: nginx`<br>`state: restarted` | handlers/main.yml | YAML dictionary syntax |
| `name=nginx state=reloaded` | `name: nginx`<br>`state: reloaded` | handlers/main.yml | YAML dictionary syntax |
| `iteritems()` | `items()` | templates/nginx.conf.j2, templates/site.j2 | Python 3 compatibility |
| `"on"` | `true` | defaults/main.yml | Boolean modernization |
| `"65"` | `65` | defaults/main.yml | Native type enforcement |
| `python-selinux` | `python3-selinux` | vars/main.yml | Python 3 compatibility |

## Dependencies

**Collection dependencies** (for requirements.yml):
- ansible.builtin: 2.13.0

**Role dependencies**: None (from meta/main.yml)

**External packages**:
- RedHat: libselinux-python, nginx
- Debian: python-selinux (to be updated to python3-selinux), nginx

**Services managed**: nginx

## Template Modernization

- **nginx.conf.j2**:
  - Replace `ansible_os_family` with `ansible_facts['os_family']`
  - Replace `ansible_processor_count` with `ansible_facts['processor_count']`
  - Replace `iteritems()` with `items()` for Python 3 compatibility
  - Update SSL protocols to remove TLSv1 and TLSv1.1, use only TLSv1.2 and TLSv1.3
  - Parameterize SSL protocols as a variable for easier management

- **site.j2**:
  - Replace `iteritems()` with `items()` for Python 3 compatibility
  - Replace string comparison using `find` method with `in` operator or `startswith()`
  - Simplify `nginx_separate_logs_per_site == True` to just `nginx_separate_logs_per_site`

- **default.conf.j2** and **default.j2**:
  - Ensure all variables are properly quoted with `{{ }}`

## Argument Specification

```yaml
# meta/argument_specs.yml
---
argument_specs:
  main:
    short_description: Install and configure Nginx web server
    description: Install and configure Nginx web server on RedHat and Debian-based systems
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

**Services to check**: nginx

**Templates to validate**:
- roles/nginx/templates/nginx.conf.j2
- roles/nginx/templates/site.j2
- roles/nginx/templates/default.conf.j2
- roles/nginx/templates/default.j2

## Pre-flight checks:
- Verify Nginx configuration syntax: `nginx -t`
- Check Nginx service status: `systemctl status nginx`
- Verify site configurations: `ls -la /etc/nginx/sites-enabled/`
- Test HTTP response: `curl -I http://localhost`
- Check for SELinux issues: `audit2allow -a`
- Verify log file permissions: `ls -la /var/log/nginx/`
- Check for open ports: `ss -tulpn | grep nginx`