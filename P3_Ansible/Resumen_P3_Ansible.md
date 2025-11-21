# Ansible

# Instalando Ansible en Alma

[https://docs.ansible.com/projects/ansible/latest/installation_guide/installation_distros.html#installing-ansible-on-debian](https://docs.ansible.com/projects/ansible/latest/installation_guide/installation_distros.html#installing-ansible-on-debian)

Por lo que leí en el enlace, parecía que era tan sencillo como:

```bash
sudo dnf update
sudo dnf install -y epel-release
sudo dnf install -y ansible

ansible --version
## Tenemos la 2.16.3 :)
```

# Configurando Ansible (inventarios)

[https://docs.ansible.com/projects/ansible/latest/inventory_guide/intro_inventory.html](https://docs.ansible.com/projects/ansible/latest/inventory_guide/intro_inventory.html)

Vamos a `/etc/ansible/hosts` e incluimos los dos servidores (puede ser en formato INI o YAML) :

```bash
[servers]
192.168.56.105 
ansible_port=22022 
ansible_user=asier 
ansible_password='practicas,ise'

192.168.56.110 
ansible_port=22022 
ansible_user=asierRocky 
ansible_password='practicas,ise'
```

El puerto, usuario y contraseña debe ser el necesario para conectarse por SSH a los servidores.

(También podríamos haber usado SSH sin contraseña, que es lo recomendado)

# Probando Ansible

### Ejecutando comandos

[https://docs.ansible.com/projects/ansible/latest/collections/ansible/builtin/ping_module.html](https://docs.ansible.com/projects/ansible/latest/collections/ansible/builtin/ping_module.html)

Tan sencillo como:

```bash
ansible all -m ping
```

Salida esperada:

```bash
[asier@localhost ~]$ ansible all -m ping
192.168.56.105 | SUCCESS => {
    "ansible_facts": {
        "discovered_interpreter_python": "/usr/bin/python3"
    },
    "changed": false,
    "ping": "pong"
}
192.168.56.110 | SUCCESS => {
    "ansible_facts": {
        "discovered_interpreter_python": "/usr/libexec/platform-python"
    },
    "changed": false,
    "ping": "pong"
}
```

→ Tuvimos un pequeño problema :

```bash
192.168.56.110 | FAILED! => {
    "msg": "Using a SSH password instead of a key is not possible because 
    Host Key checking is enabled and sshpass does not support this.  
    Please add this host's fingerprint to your known_hosts file to manage this host."
}

##Solucionado con:
ssh-keyscan -p 22022 192.168.56.110 >> ~/.ssh/known_hosts

#Se ve que usábamos otro archivo para conectarnos a Alma que no era known_hosts,
#que es lo que usa Ansible
```

### Ejecutando scripts

[https://docs.ansible.com/projects/ansible/latest/collections/ansible/builtin/script_module.html](https://docs.ansible.com/projects/ansible/latest/collections/ansible/builtin/script_module.html)

[https://labex.io/tutorials/ansible-how-to-execute-a-script-on-a-remote-host-with-ansible-414953](https://labex.io/tutorials/ansible-how-to-execute-a-script-on-a-remote-host-with-ansible-414953)

[https://docs.ansible.com/projects/ansible/latest/playbook_guide/index.html](https://docs.ansible.com/projects/ansible/latest/playbook_guide/index.html)

Tan sencillo como basarse en este ejemplo de uso del módulo script:

```bash
name: Run a script with arguments (free form)
  ansible.builtin.script: /some/local/script.sh --some-argument 1234
  
## También se puede usar script en lugar de ansible.builtin.script ...

name: Script de monitorizacion de RAID1
  hosts: servers          
  tasks:
    - name: Ejecutar script de monitorizacion RAID
      script: /etc/ansible/mon_raid.py

```

Para ejecutarlo, basta con el mítico:

```bash
ansible-playbook mon_raid.yaml 
```

Obtenemos la salida esperada (no creo que haga falta usar `register` o `debug` para verla):

```bash
[asier@localhost ansible]$ ansible-playbook mon_raid.yaml 

PLAY [Script de monitorizacion de RAID1] ***************************************************************************************************************

TASK [Gathering Facts] *********************************************************************************************************************************
ok: [192.168.56.105]
ok: [192.168.56.110]

TASK [Ejecutar script de monitorizacion RAID] **********************************************************************************************************
changed: [192.168.56.105]
changed: [192.168.56.110]

PLAY RECAP *********************************************************************************************************************************************
192.168.56.105             : ok=2    changed=1    unreachable=0    failed=0    skipped=0    rescued=0    ignored=0   
192.168.56.110             : ok=2    changed=1    unreachable=0    failed=0    skipped=0    rescued=0    ignored=0   

```

### Ejecutando playbooks

Vamos a ver si encontramos un `playbook` para comprobar la versión de httpd …

[https://docs.redhat.com/es/documentation/red_hat_ansible_automation_platform/2.5/html/getting_started_with_playbooks/assembly-playbook-practical-example](https://docs.redhat.com/es/documentation/red_hat_ansible_automation_platform/2.5/html/getting_started_with_playbooks/assembly-playbook-practical-example)

[https://stackoverflow.com/questions/74970195/creating-a-ansible-playbook-for-getting-the-apache-status-and-want-to-display-ju](https://stackoverflow.com/questions/74970195/creating-a-ansible-playbook-for-getting-the-apache-status-and-want-to-display-ju)

Encontrado! Vamos a adaptarlo a nuestro gusto, no sin antes entender que hace:

[https://docs.ansible.com/projects/ansible/latest/collections/ansible/builtin/dnf_module.html](https://docs.ansible.com/projects/ansible/latest/collections/ansible/builtin/dnf_module.html)

```bash
- name: Checkear apache en Alma Linux
  hosts: servers
  become: true

  tasks:
    - name: Ensure apache is at the latest version
      ansible.builtin.dnf:
        name: httpd
        state: latest
```

*Nótese que si usamos `dnf` y `Apache` solo nos servirá para Alma, y viceversa con `apt` y `httpd` para Debian. Por simplicidad, usaré un playbook para cada uno.* 

*Usaremos `register` esta vez en este ejemplo, por amor al arte.*

`Playbook` final después de mucho ensayo y error:

```bash
- name: Instalar y comprobar Apache en servidor Debian
  hosts: 192.168.56.105
  become: true
  tasks:

    - name: Comprobar la version de apache
      ansible.builtin.apt:
        name: apache2
        state: latest

    - name: Comprobar status de apache
      ansible.builtin.service:
        name: apache2
        state: started
      register: apache2_status

    - name: Mostrar estado del servicio Apache
      debug:
        var: apache2_status.state

## hay que usar -K en ansible-playbook si usamos root

```

Con esto ponemos fin al apartado de Ansible.