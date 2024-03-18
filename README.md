# Домашнее задание к занятию 3 «Использование Ansible»   
Play, который устанавливает и настраивает LightHouse.   

```yml
- name: Install Lighthouse
  hosts: lighthouse
  tags:
    - lighthouse
    - nginx
  handlers:
    - name: Start-nginx
      become: true
      ansible.builtin.command: 
        cmd: nginx    
    - name: Reload-nginx
      become: true
      ansible.builtin.command: 
        cmd: nginx -s reload
  pre_tasks:
    - name: install git
      become: true
      ansible.builtin.yum:
        name: git
        state: present
    - name: install epel-release
      become: true
      ansible.builtin.yum:
        name: epel-release
        state: present   
  tasks:
    - name: install NGINX
      become: true
      ansible.builtin.yum:
        name: nginx
        state: present
      notify: Start-nginx
    - name: Flush handlers
      ansible.builtin.meta: flush_handlers  
    - name: Apply nginx config
      become: true
      ansible.builtin.template:
        src: templates/nginx.conf.j2
        dest: /etc/nginx/nginx.conf
        mode: "0777"
      notify: Reload-nginx  
    - name: Lighthouse | Copy from git
      become: true
      ansible.builtin.git:
        repo: "{{ lighthouse_vcs }}"
        version: master
        dest: "{{ lighthouse_local_dir }}"
    - name: Lighthouse | Create ligthouse config
      become: true
      ansible.builtin.template:
        src: lighthouse.conf.j2
        dest: /etc/nginx/conf.d/defult.conf
        mode: "0777"
      notify: Reload-nginx
    - name: Print address to connect
      ansible.builtin.debug:
        msg: "Lighthouse is available at http://{{ ansible_host }}:{{ lighthouse_port }}/"
```   

prod.yml:
```yml
---
clickhouse:
  hosts:
    clickhouse-01:
      ansible_host: 158.160.48.191
      #ansible_user: ansible
      #ansible_connection: ssh
vector:
  hosts:
    vector-01:
      ansible_host: 158.160.42.220
      #ansible_user: vector
      #ansible_connection: ssh
lighthouse:
  hosts:
    lighthouse-01:
      ansible_host: 158.160.34.167
      ansible_user: lighthouse
      #ansible_connection: ssh
```   
```bash
user@study:~/home_work/ansible/ansible-03-yandex$ ansible-lint playbook/site.yml
WARNING  Listing 3 violation(s) that are fatal
name[missing]: All tasks should be named.
playbook/site.yml:12 Task/Handler: block/always/rescue 

no-changed-when: Commands should not change things if nothing needs doing.
playbook/site.yml:76 Task/Handler: Start-nginx

no-changed-when: Commands should not change things if nothing needs doing.
playbook/site.yml:80 Task/Handler: Reload-nginx

Read documentation for instructions on how to ignore specific rule violations.

                  Rule Violation Summary                  
 count tag             profile rule associated tags       
     1 name[missing]   basic   idiom                      
     2 no-changed-when shared  command-shell, idempotency 

Failed: 3 failure(s), 0 warning(s) on 1 files. Last profile that met the validation criteria was 'min'.
user@study:~/home_work/ansible/ansible-03-yandex$ 
```   
--check:
```bash
user@study:~/home_work/ansible/ansible-03-yandex$ ansible-playbook -i ./playbook/inventory/prod.yml ./playbook/site.yml --check

PLAY [Install Clickhouse] ****************************************************************************************************************************************************************************************************

TASK [Gathering Facts] *******************************************************************************************************************************************************************************************************
ok: [clickhouse-01]

TASK [Get clickhouse distrib 1] **********************************************************************************************************************************************************************************************
ok: [clickhouse-01] => (item=clickhouse-client)
ok: [clickhouse-01] => (item=clickhouse-server)
failed: [clickhouse-01] (item=clickhouse-common-static) => {"ansible_loop_var": "item", "changed": false, "dest": "./clickhouse-common-static-22.3.3.44.rpm", "elapsed": 0, "gid": 1000, "group": "user", "item": "clickhouse-common-static", "mode": "0777", "msg": "Request failed", "owner": "user", "response": "HTTP Error 404: Not Found", "secontext": "unconfined_u:object_r:user_home_t:s0", "size": 246310036, "state": "file", "status_code": 404, "uid": 1000, "url": "https://packages.clickhouse.com/rpm/stable/clickhouse-common-static-22.3.3.44.noarch.rpm"}

TASK [Get clickhouse distrib 2] **********************************************************************************************************************************************************************************************
ok: [clickhouse-01]

TASK [Install clickhouse packages] *******************************************************************************************************************************************************************************************
ok: [clickhouse-01]

TASK [Flush handlers] ********************************************************************************************************************************************************************************************************

TASK [Create database] *******************************************************************************************************************************************************************************************************
skipping: [clickhouse-01]

PLAY [Install Vector] ********************************************************************************************************************************************************************************************************

TASK [Gathering Facts] *******************************************************************************************************************************************************************************************************
ok: [vector-01]

TASK [Get Vector distrib] ****************************************************************************************************************************************************************************************************
ok: [vector-01]

TASK [Install vector packages] ***********************************************************************************************************************************************************************************************
ok: [vector-01]

TASK [Config template] *******************************************************************************************************************************************************************************************************
ok: [vector-01]

PLAY [Install Lighthouse] ****************************************************************************************************************************************************************************************************

TASK [Gathering Facts] *******************************************************************************************************************************************************************************************************
ok: [lighthouse-01]

TASK [Install git] ***********************************************************************************************************************************************************************************************************
ok: [lighthouse-01]

TASK [Install epel-release] **************************************************************************************************************************************************************************************************
ok: [lighthouse-01]

TASK [Install Nginx] *********************************************************************************************************************************************************************************************************
ok: [lighthouse-01]

TASK [Flush handlers] ********************************************************************************************************************************************************************************************************

TASK [Apply nginx config] ****************************************************************************************************************************************************************************************************
ok: [lighthouse-01]

TASK [Lighthouse | Copy from git] ********************************************************************************************************************************************************************************************
ok: [lighthouse-01]

TASK [Lighthouse | Create ligthouse config] **********************************************************************************************************************************************************************************
ok: [lighthouse-01]

TASK [Print address to connect] **********************************************************************************************************************************************************************************************
ok: [lighthouse-01] => {
    "msg": "Lighthouse is available at http://158.160.34.167:80/"
}

PLAY RECAP *******************************************************************************************************************************************************************************************************************
clickhouse-01              : ok=3    changed=0    unreachable=0    failed=0    skipped=1    rescued=1    ignored=0   
lighthouse-01              : ok=8    changed=0    unreachable=0    failed=0    skipped=0    rescued=0    ignored=0   
vector-01                  : ok=4    changed=0    unreachable=0    failed=0    skipped=0    rescued=0    ignored=0   
```   
--diff:   
```bash
PLAY RECAP *******************************************************************************************************************************************************************************************************************
clickhouse-01              : ok=4    changed=0    unreachable=0    failed=0    skipped=0    rescued=1    ignored=0   
lighthouse-01              : ok=8    changed=0    unreachable=0    failed=0    skipped=0    rescued=0    ignored=0   
vector-01                  : ok=4    changed=0    unreachable=0    failed=0    skipped=0    rescued=0    ignored=0   
```   
Описание Playbook: https://github.com/suntsovvv/ansible-03-yandex/blob/main/playbook/README.md

