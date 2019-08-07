# KubPi : déploiement d'un cluster k8s sur raspberry

Structure de Hosts.yml

```
all:
  vars:
    ip_master: 192.168.1.25
  children:
    nodes:
      hosts:
        pi1:
          ansible_host: 192.168.1.20
        pi2:
          ansible_host: 192.168.1.21
        pi3:
          ansible_host: 192.168.1.22
        pi4:
          ansible_host: 192.168.1.23
        pi5:
          ansible_host: 192.168.1.24
        pi6:
          ansible_host: 192.168.1.25
          master: true
```

Playbook d'installation

```
- hosts: master
  roles:
    - kubpi
```

Création du cluster

```
ansible-playbook -i hosts.yml -u pi -k -b install-kub.yml
```

Reset all

```
ansible-playbook -i hosts.yml -u pi -k -b reset-all.yml
```

Playbook de reset du cluster

```
- hosts: all
  become: yes
  tasks:

  - name: kubeadm reset
    shell: kubeadm reset -f

  - name: check clean docker
    shell: docker ps -aq | wc -l
    register: docker_container_exist

  - name: clean docker
    shell: docker rm -f $(docker ps -aq)
    when: docker_container_exist.stdout != "0"

  - name: suppression paquet kubernetes
    apt:
      name:
       - kubectl
       - kubadm
       - kubernetes-cni
       - kubelet
      purge: yes
      autoremove: yes
      state: absent
```

