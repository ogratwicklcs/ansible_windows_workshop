Although it is possible to write a playbook in one file as we’ve done
throughout this workshop, eventually you’ll want to reuse files and
start to organize things.

Section 1: Create directory structure for your new role
=======================================================

Step 1:
-------

In Visual Studio Code, navigate to the Terminal drop-down menu at the top and select "New Terminal"

This will open a terminal with bash that will allow you to run commands on the Ansible Tower control host.  We will use this terminal to create our role folder structure.


![terminal menu](images/terminal.PNG)

Step 2:
-------

Create folder structure for your first role underneath the iis_advanced directory from with bash commands from the terminal.

```bash
[studentX@ansible ~]$ cd ~/windows-workshop/workshop_project/iis_advanced
[studentX@ansible iis_advanced]$ mkdir roles
[studentX@ansible roles]$ cd roles
[studentX@ansible roles]$ mkdir iis_simple
[studentX@ansible roles]$ cd iis_simple
[studentX@ansible iis_simple]$ mkdir defaults vars handlers tasks templates
```

Step 3:
-------

Within each of these new directories (except templates), right-click and
create *New File* Create a file called `main.yml` in each of these
directories. You will not do this under templates as this is where we will create
individual template files. 

The finished structure will look like this:

![Role Structure](images/6-create-role.png)

Section 2: Breaking Your `site.yml` Playbook into the Newly Created `iis_simple` Role
=====================================================================================

In this section, we will separate out the major parts of your playbook
including `vars:`, `tasks:`, `template:`, and `handlers:`.

Step 1:
-------

Make a backup copy of `site.yml`, then create a new `site.yml`.

Navigate to your `iis_advanced` folder, right click `site.yml`, click
`rename`, and call it `site.yml.backup`


Step 2:
-------

Update site.yml.  It should look like
below:

```yaml
---
- hosts: windows
  name: This is my role-based playbook

  roles:
    - iis_simple
```

![New site.yml](images/6-new-site.png)

Step 3:
-------

Add a default variable to your role. Edit the
`roles\iis_simple\defaults\main.yml` as follows:

```yaml
---
# defaults file for iis_simple
iis_sites:
  - name: 'Ansible Playbook Test'
    port: '8080'
    path: 'C:\sites\playbooktest'
  - name: 'Ansible Playbook Test 2'
    port: '8081'
    path: 'C:\sites\playbooktest2'
```

Step 4:
-------

Add some role-specific variables to your role in
`roles\iis_simple\vars\main.yml`.

```yaml
---
# vars file for iis_simple
iis_test_message: "Hello World!  My test IIS Server"
```

Step 5:
-------

Create your role handler in `roles\iis_simple\handlers\main.yml`.

```yaml
---
# handlers file for iis_simple
- name: restart iis service
  win_service:
    name: W3Svc
    state: restarted
    start_mode: auto
```

Step 6:
-------

Add tasks to your role in `roles\iis_simple\tasks\main.yml`.

<!-- {% raw %} -->
```yaml
---
# tasks file for iis_simple

- name: Install IIS
  win_feature:
    name: Web-Server
    state: present

- name: Create site directory structure
  win_file:
    path: "{{ item.path }}"
    state: directory
  with_items: "{{ iis_sites }}"

- name: Create IIS site
  win_iis_website:
    name: "{{ item.name }}"
    state: started
    port: "{{ item.port }}"
    physical_path: "{{ item.path }}"
  with_items: "{{ iis_sites }}"
  notify: restart iis service

- name: Open port for site on the firewall
  win_firewall_rule:
    name: "iisport{{ item.port }}"
    enable: yes
    state: present
    localport: "{{ item.port }}"
    action: Allow
    direction: In
    protocol: Tcp
  with_items: "{{ iis_sites }}"

- name: Template simple web site to iis_site_path as index.html
  win_template:
    src: 'index.html.j2'
    dest: '{{ item.path }}\index.html'
  with_items: "{{ iis_sites }}"

- name: Show website addresses
  debug:
    msg: "{{ item }}"
  loop:
    - http://{{ ansible_host }}:8080
    - http://{{ ansible_host }}:8081
```
<!-- {% endraw %} -->

Step 7:
-------

Add your index.html template.

Right-click `roles\iis_simple\templates` and create a new file called
`index.html.j2` with the following content:

<!-- {% raw %} -->
```html
<html>
<body>

  <p align=center><img src='http://docs.ansible.com/images/logo.png' align=center>
  <h1 align=center>{{ ansible_hostname }} --- {{ iis_test_message }}

</body>
</html>
```
<!-- {% endraw %} -->

Now, remember we still have a *templates* folder at the base level of
this playbook, so we will delete that now. Right click it and Select
*Delete*.

Step 8: Commit
--------------

Click the Source Code icon as shown below (1).

Type in a commit message like `Adding iis_simple role` (2) and click the
check box above (3).

![Commit iis\_simple\_role](images/6-commit.png)

Click the `synchronize changes` button on the blue bar at the bottom
left. This should again return with no problems.

Section 3: Running your new playbook
====================================

Now that you’ve successfully separated your original playbook into a
role, let’s run it and see how it works. We don’t need to create a new
template, as we are re-using the one from Exercise 5. When we run the
template again, it will automatically refresh from git and launch our
new role.

Step 1:
-------

Before we can modify our Job Template, you must first go resync your
Project again. So do that now.

Step 2:
-------

Select TEMPLATES

> **Note**
>
> Alternatively, if you haven’t navigated away from the job templates
> creation page, you can scroll down to see all existing job templates

Step 3:
-------

Click the rocketship icon ![Add](images/at_launch_icon.png) for the
**IIS Advanced** Job Template.

Step 4:
-------

When prompted, enter your desired test message

If successful, your standard output should look similar to the figure
below. Note that most of the tasks return OK because we’ve previously
configured the servers and services are already running.

![Job output](images/6-job-output.png)


When the job has successfully completed, you should see two URLs to your websites printed at the bottom of the job output. Verify they are still working.

Section 5: Review
=================

You should now have a completed playbook, `site.yml` with a single role
called `iis_simple`. The advantage of structuring your playbook into
roles is that you can now add reusability to your playbooks as well as
simplifying changes to variables, tasks, templates, etc.

[Ansible Galaxy](https://galaxy.ansible.com) is a good repository of
roles for use or reference.

----
**Navigation**
<br>
[Previous Exercise](../5-adv-playbook) - [Next Exercise](../7-win-patch)

