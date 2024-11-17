# Ansible

## 0. Prerequirements

* Install ansible and openssh-server

    ```bash
    sudo apt-get update
    sudo apt-get install ansible openssh-server -y
    ```

* Copy ssh key to localhost and client's machine

    ```bash
    ssh-copy-id localhost
    ssh-copy-id 192.168.122.20
    ```

## 1. Inventory file configuration

* Edit *hosts* file

    ```bash
    sudo nano /etc/ansible/hosts
    ```

* Specify hosts addresses

    ```bash
    [server]
    192.168.122.13

    [clients]
    192.168.122.20
    ```

    Or their domain names

    ```bash
    [server]
    srv.iav.miet.stu

    [clients]
    cli1.iav.miet.stu
    ```

## 2. Let's check some ansible commands

Options:

* `-a` - Run command on dedicated host

* `-b`, `--become` - Run with SU

* `-K` - SU password reqired

```bash
# Run command on dedicated hosts with -a option
ansible -a 'cat /etc/astra_version' all
```

Output message:

```bash
astra@server:~$ ansible  -a 'cat /etc/astra_version' all
srv.iav.miet.stu | CHANGED | rc=0 >>
1.7.2

cli1.iav.miet.stu | CHANGED | rc=0 >>
1.7.2
```

## 3. Playbooks

### 3.1 Playbook example #1

Print value of free RAM for server and client

* [freeRAM.yml](scripts/freeRAM.yml) playbook

    ```yml
    - name: freeRAM
      hosts: all
      become: false
      tasks:
        - name: findRAM
          ansible.builtin.shell:
            cmd: cat /proc/meminfo | grep "MemFree:"
          register: meminfo_out
        - name: showRAM
          debug:
            msg: "Free RAM: {{meminfo_out.stdout}}"
    ```

* Command:

    ```bash
    ansible-playbook ./freeRAM.yml
    ```

* Result:

    ```bash
    PLAY [freeRAM] *****************************************************************

    TASK [Gathering Facts] *********************************************************
    ok: [cli1.iav.miet.stu]
    ok: [srv.iav.miet.stu]

    TASK [findRAM] *****************************************************************
    changed: [cli1.iav.miet.stu]
    changed: [srv.iav.miet.stu]

    TASK [showRAM] *****************************************************************
    ok: [srv.iav.miet.stu] => {
        "msg": "Free RAM: MemFree:         3011836 kB"
    }
    ok: [cli1.iav.miet.stu] => {
        "msg": "Free RAM: MemFree:         3141840 kB"
    }

    PLAY RECAP *********************************************************************
    cli1.iav.miet.stu          : ok=3    changed=1    unreachable=0    failed=0    skipped=0    rescued=0    ignored=0   
    srv.iav.miet.stu           : ok=3    changed=1    unreachable=0    failed=0    skipped=0    rescued=0    ignored=0 
    ```

### 3.2 Playbook example #2

Entire solution: [system_info.yml](scripts/system_info.yml)

* Playbook header

    ```yml
    - name: Collect System Info Playbook
      hosts:
        - srv.iav.miet.stu
        - cli1.iav.miet.stu
      become: no
      vars:
        initials: "iav"
        server_host: "srv.iav.miet.stu"
        client_host: "cli1.iav.miet.stu"
        log_fname: "info.log"
        server_dir_name: "Server{{ initials }}"
        client_dir_name: "Client{{ initials }}"
        server_dir_path: "{{ ansible_env.HOME }}/{{server_dir_name}}"
        client_dir_path: "{{ ansible_env.HOME }}/{{client_dir_name}}"
        server_log_file_path: "{{ server_dir_path }}/{{ log_fname }}"
        client_log_file_path: "{{ client_dir_path }}/{{ log_fname }}"
        temp_fname: "temp.log"
        temp_fpath: "{{ ansible_env.HOME }}/{{temp_fname}}"
      gather_facts: true
      ```

* Create directories ~/ServerIAV on the server and ~/ClientIAV on the client.

    ```yml
    tasks:
    - name: Create dir on server
        file:
        path: "{{ server_dir_path }}"
        state: directory
        when: inventory_hostname == server_host

    - name: Create dir on client
        file:
        path: "{{ client_dir_path }}"
        state: directory
        when: inventory_hostname == client_host
    ```

* Create *info* files in home dirs. Add a check for the existence of the file.

    ```yml
        - name: Checking if the file exists on server
            stat:
            path: "{{ server_log_file_path }}"
            when: inventory_hostname == server_host
            register: srv_file_check

        - name: Checking if the file exists on client
            stat:
            path: "{{ client_log_file_path }}"
            when: inventory_hostname == client_host
            register: cli_file_check

        - name: Create system info file on server
            file:
            path: "{{ server_log_file_path }}"
            state: touch
            when: inventory_hostname == server_host and srv_file_check.stat.exists == false

        - name: Create system info file on client
            file:
            path: "{{ client_log_file_path }}"
            state: touch
            when: inventory_hostname == client_host and cli_file_check.stat.exists == false
    ```

* Fill this files with the folowwing information: hostname, last name, ip adress, RAM size in Mb, average load for the last 15 minutes of work (use /proc/loadavg file).

    Format: hostname | username | 192.168.122.1 | 722 | 1.58

    ```yml
    - name: Get IP address
        ansible.builtin.command: hostname -I
        register: ip_address
        changed_when: false

    - name: Get memory usage in MB
        ansible.builtin.shell: "free -m | awk '/Mem:/ {print $3}'"
        register: memory_usage
        changed_when: false

    - name: Get average load in last 15 minutes
        ansible.builtin.shell: "awk '{print $3}' /proc/loadavg"
        register: load_average
        changed_when: false

* Edit username in copied files.

    ```yml
    - name: Write metrics to temporary file
        ansible.builtin.lineinfile:
        path: "{{ ansible_env.HOME }}/{{temp_fname}}"
        create: true
        line: "{{ ansible_hostname }} | Bebopovsky | {{ ip_address.stdout.strip() }} | {{ memory_usage.stdout.strip() }} | {{ load_average.stdout.strip() }}"
        become: false

    - name: Debug
        ansible.builtin.debug:
        msg: "Target directory: {{ client_dir_path }}' if inventory_hostname == '{{ client_host }}' else '{{ server_dir_name }}' }}"
    ```

* Copy created file to ~/{Server | Client}IAV.

    ```yml
    - name: Move file to directories
        ansible.builtin.command: >
        mv "{{ ansible_env.HOME }}/{{ temp_fname }}" "{{ server_log_file_path if inventory_hostname == server_host else client_log_file_path }}"
        become: false

    - name: Delete temporary file
        ansible.builtin.file:
        path: "{{ ansible_env.HOME }}/{{temp_fname}}"
        state: absent
    ```

* If load is bigger than 1: state NAME_MACHINE bad. Else: state NAME_MACHINE good

    ```yml
    - name: Check average load and display status message
        ansible.builtin.debug:
        msg: "state {{ ansible_hostname }} {{ 'High load in last 15 min' if load_average.stdout | float > 1 else 'Low load in last 15 min' }}"
    ```

* Run playbook:

    ```bash
    ansible-playbook ./system_info.yml
    ```

Results:

```bash
PLAY [Collect System Info Playbook] ********************************************

TASK [Gathering Facts] *********************************************************
ok: [srv.iav.miet.stu]
ok: [cli1.iav.miet.stu]

TASK [Create dir on server] ****************************************************
skipping: [cli1.iav.miet.stu]
ok: [srv.iav.miet.stu]

TASK [Create dir on client] ****************************************************
skipping: [srv.iav.miet.stu]
ok: [cli1.iav.miet.stu]

TASK [Checking if the file exists on server] ***********************************
skipping: [cli1.iav.miet.stu]
ok: [srv.iav.miet.stu]

TASK [Checking if the file exists on client] ***********************************
skipping: [srv.iav.miet.stu]
ok: [cli1.iav.miet.stu]

TASK [Create system info file on server] ***************************************
skipping: [srv.iav.miet.stu]
skipping: [cli1.iav.miet.stu]

TASK [Create system info file on client] ***************************************
skipping: [srv.iav.miet.stu]
skipping: [cli1.iav.miet.stu]

TASK [Get IP address] **********************************************************
ok: [cli1.iav.miet.stu]
ok: [srv.iav.miet.stu]

TASK [Get memory usage in MB] **************************************************
ok: [cli1.iav.miet.stu]
ok: [srv.iav.miet.stu]

TASK [Get average load in last 15 minutes] *************************************
ok: [cli1.iav.miet.stu]
ok: [srv.iav.miet.stu]

TASK [Write metrics to temporary file] *****************************************
changed: [cli1.iav.miet.stu]
changed: [srv.iav.miet.stu]

TASK [Debug] *******************************************************************
ok: [srv.iav.miet.stu] => {
    "msg": "Target directory: /home/astra/Clientiav' if inventory_hostname == 'cli1.iav.miet.stu' else 'Serveriav' }}"                            
}
ok: [cli1.iav.miet.stu] => {
    "msg": "Target directory: /home/astra/Clientiav' if inventory_hostname == 'cli1.iav.miet.stu' else 'Serveriav' }}"
}

TASK [Move file to directories] ************************************************
changed: [srv.iav.miet.stu]
changed: [cli1.iav.miet.stu]

TASK [Delete temporary file] ***************************************************
ok: [cli1.iav.miet.stu]
ok: [srv.iav.miet.stu]

TASK [Check average load and display status message] ***************************
ok: [srv.iav.miet.stu] => {
    "msg": "state server Low load in last 15 min"
}
ok: [cli1.iav.miet.stu] => {
    "msg": "state client Low load in last 15 min"
}

PLAY RECAP *********************************************************************
cli1.iav.miet.stu          : ok=11   changed=2    unreachable=0    failed=0    skipped=4    rescued=0    ignored=0   
srv.iav.miet.stu           : ok=11   changed=2    unreachable=0    failed=0    skipped=4    rescued=0    ignored=0
```

Check saved logs:

```bash
astra@server:~$ cat Serveriav/info.log
server | Bebopovsky | 10.0.2.15 192.168.122.13 fd00::a00:27ff:fe05:973d | 519 | 0.05
```

```bash
astra@client:~$ cat Clientiav/info.log 
client | Bebopovsky | 10.0.2.15 192.168.122.20 fd00::a00:27ff:fe5f:9739 | 383 | 0.03
```
