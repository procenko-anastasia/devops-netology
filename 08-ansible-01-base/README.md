# Домашнее задание к занятию "08.01 Введение в Ansible"

## Проценко Анастасия

## Подготовка к выполнению

----
1. *Установите ansible версии 2.10 или выше.*  
  
```bash
Anastasia@MacBook-Air ~ % ansible --version
ansible [core 2.12.6]
  config file = None
  configured module search path = ['/Users/Anastasia/.ansible/plugins/modules', '/usr/share/ansible/plugins/modules']
  ansible python module location = /opt/homebrew/Cellar/ansible/5.9.0/libexec/lib/python3.10/site-packages/ansible
  ansible collection location = /Users/Anastasia/.ansible/collections:/usr/share/ansible/collections
  executable location = /opt/homebrew/bin/ansible
  python version = 3.10.4 (main, Apr 26 2022, 19:36:29) [Clang 13.1.6 (clang-1316.0.21.2)]
  jinja version = 3.1.2
  libyaml = True
```

----
2. *Создайте свой собственный публичный репозиторий на github с произвольным именем.*  

**Выполнено: https://github.com/procenko-anastasia/netology8**

----
3. *Скачайте [playbook](./playbook/) из репозитория с домашним заданием и перенесите его в свой репозиторий.*  

**Выполнено**

----

## Основная часть
1. *Попробуйте запустить playbook на окружении из `test.yml`, зафиксируйте какое значение имеет факт `some_fact` для указанного хоста при выполнении playbook'a.*  

```Anastasia@MacBook-Air playbook % ansible-playbook -i inventory/test.yml site.yml 

PLAY [Print os facts] ***************************************************************************************************************************************************

TASK [Gathering Facts] **************************************************************************************************************************************************
ok: [localhost]

TASK [Print OS] *********************************************************************************************************************************************************
ok: [localhost] => {
    "msg": "MacOSX"
}

TASK [Print fact] *******************************************************************************************************************************************************
ok: [localhost] => {
    "msg": 12
}

PLAY RECAP **************************************************************************************************************************************************************
localhost                  : ok=3    changed=0    unreachable=0    failed=0    skipped=0    rescued=0    ignored=0   
```

**`some_fact`=12**

----
2. *Найдите файл с переменными (group_vars) в котором задаётся найденное в первом пункте значение и поменяйте его на 'all default fact'.*  

**Изменяем содержимое переменной `some_fact` в файле `group_vars/all/examp.yml` на `all default fact`**  
**Проверяем:**  
```
Anastasia@MacBook-Air playbook % ansible-playbook -i inventory/test.yml site.yml 

PLAY [Print os facts] ********************************************************************************************************************************************

TASK [Gathering Facts] *******************************************************************************************************************************************
ok: [localhost]

TASK [Print OS] **************************************************************************************************************************************************
ok: [localhost] => {
    "msg": "MacOSX"
}

TASK [Print fact] ************************************************************************************************************************************************
ok: [localhost] => {
    "msg": "all default fact"
}

PLAY RECAP *******************************************************************************************************************************************************
localhost                  : ok=3    changed=0    unreachable=0    failed=0    skipped=0    rescued=0    ignored=0 
```

----
3. *Воспользуйтесь подготовленным (используется `docker`) или создайте собственное окружение для проведения дальнейших испытаний.*  

**Скачиваем docker-образы `CentOS7` и `Ubuntu`:**  
```bash
docker login
docker run centos:centos7
docker run pycontribs/ubuntu:latest
```

**Проверяем**
```bash
Anastasia@MacBook-Air ~ % docker images                                                      
REPOSITORY               TAG         IMAGE ID       CREATED        SIZE
pycontribs/ubuntu        latest      42a4e3b21923   2 years ago    664MB
adminer                  latest      ca86d5f7e7fa   2 months ago   80.6MB
postgres                 12-alpine   dbc283fda80c   2 months ago   197MB
mysql                    8-oracle    eb3f4a2767a3   2 months ago   486MB
postgres                 12          ee0f696f99b2   2 months ago   352MB
postgres                 latest      6377d4b43c30   2 months ago   355MB
docker/getting-started   latest      adfdb308d623   4 months ago   27.4MB
centos                   centos7     eeb6ee3f44bd   9 months ago   204MB
```

**Запускаем и проверяем**
```bash
Anastasia@MacBook-Air ~ % docker run -d --name centos7 eeb6ee3f44bd /bin/sleep 100000000000000
4ef41e17475d6bf4e6bb8f474aae1a8fd52d2f13879bfe4c7fc37e0ae525f512

Anastasia@MacBook-Air playbook % docker run -d --name ubuntu 42a4e3b21923 /bin/sleep 100000000000000 
2ad6876f22d5be2b737c186f7781e972172e0d09ab9c1f33e7b6b57c99b37217

Anastasia@MacBook-Air ~ % docker ps -a                                                       
CONTAINER ID   IMAGE            COMMAND                  CREATED         STATUS                   PORTS     NAMES
2ad6876f22d5   42a4e3b21923     "/bin/sleep 10000000…"   13 seconds ago  Up 12 seconds                      ubuntu
4ef41e17475d   eeb6ee3f44bd     "/bin/sleep 10000000…"   4 minutes ago   Up 4 minutes                       centos7
a7273c4b64ed   adminer          "entrypoint.sh docke…"   2 months ago    Exited (0) 2 hours ago             netology-adminer-1
6dc3a7ba6f2c   mysql:8-oracle   "docker-entrypoint.s…"   2 months ago    Exited (0) 2 hours ago             netology-db-1
```

----
4. *Проведите запуск playbook на окружении из `prod.yml`. Зафиксируйте полученные значения `some_fact` для каждого из `managed host`.*  

**Проверяем:**  
```
PLAY [Print os facts] *****************************************************************************************************************************

TASK [Gathering Facts] ****************************************************************************************************************************
ok: [ubuntu]
ok: [centos7]

TASK [Print OS] ***********************************************************************************************************************************
ok: [centos7] => {
    "msg": "CentOS"
}
ok: [ubuntu] => {
    "msg": "Ubuntu"
}

TASK [Print fact] *********************************************************************************************************************************
ok: [centos7] => {
    "msg": "el"
}
ok: [ubuntu] => {
    "msg": "deb"
}

PLAY RECAP ****************************************************************************************************************************************
centos7                    : ok=3    changed=0    unreachable=0    failed=0    skipped=0    rescued=0    ignored=0   
ubuntu                     : ok=3    changed=0    unreachable=0    failed=0    skipped=0    rescued=0    ignored=0   
```

- `CentOS7` -  `some_fact` = `el`
- `Ubuntu` - `some_fact` = `deb`

----
5. *Добавьте факты в `group_vars` каждой из групп хостов так, чтобы для `some_fact` получились следующие значения: для `deb` - 'deb default fact', для `el` - 'el default fact'.*  

**Для `deb` в `group_vars/deb/examp.yml` прописываем в `some_fact` значение `deb default fact`**  
**Для `el` в `group_vars/el/examp.yml` прописываем в `some_fact` значение `el default fact`**  

----
6.  *Повторите запуск playbook на окружении `prod.yml`. Убедитесь, что выдаются корректные значения для всех хостов.*

**Проверяем:**
```
Anastasia@MacBook-Air playbook % ansible-playbook -i inventory/prod.yml site.yml

PLAY [Print os facts] *****************************************************************************************************************************

TASK [Gathering Facts] ****************************************************************************************************************************
ok: [ubuntu]
ok: [centos7]

TASK [Print OS] ***********************************************************************************************************************************
ok: [centos7] => {
    "msg": "CentOS"
}
ok: [ubuntu] => {
    "msg": "Ubuntu"
}

TASK [Print fact] *********************************************************************************************************************************
ok: [centos7] => {
    "msg": "el default fact"
}
ok: [ubuntu] => {
    "msg": "deb default fact"
}

PLAY RECAP ****************************************************************************************************************************************
centos7                    : ok=3    changed=0    unreachable=0    failed=0    skipped=0    rescued=0    ignored=0   
ubuntu                     : ok=3    changed=0    unreachable=0    failed=0    skipped=0    rescued=0    ignored=0   
```

----
7. *При помощи `ansible-vault` зашифруйте факты в `group_vars/deb` и `group_vars/el` с паролем `netology`.*  

```bash
ansible-vault encrypt group_vars/el/examp.yml group_vars/deb/examp.yml
```

**Проверяем:**  
```bash
Anastasia@MacBook-Air playbook % cat group_vars/el/examp.yml
$ANSIBLE_VAULT;1.1;AES256
37663362383336653434383539353565303838653030323839636364613864336264626435653464
6564623466646433623237393564316431306232343038310a313735356535633264393464643165
36313530356532306335386434626165663134373864323264376163626166353861636634393936
6230376437356563630a323239383931386264366561623964396435303638616231643365633665
31333765306532623765373936303238623665303966393037666632316662306362373565353931
3338636231353132303138373461356231363261316337343937
Anastasia@MacBook-Air playbook % cat group_vars/deb/examp.yml
$ANSIBLE_VAULT;1.1;AES256
38643333383864363732363031383666373965333531643933336338646361353838643761373262
3566623131616333326366313036643237373964343166650a323161343864333362343333393331
61646432303639313838396437643763623762366339633936323037633661633661623033313438
6338363333386230370a336238313834313061393661393362326632653336373834383331666338
35326261356361616131616531316433633836643835306538353636663366393135613339653762
6233646231626330343261333433396138393538353962373332
```

----
8. *Запустите playbook на окружении `prod.yml`. При запуске `ansible` должен запросить у вас пароль. Убедитесь в работоспособности.*  
  
```
Anastasia@MacBook-Air playbook % ansible-playbook -i inventory/prod.yml site.yml --ask-vault-pass
Vault password: 

PLAY [Print os facts] *****************************************************************************************************************************

TASK [Gathering Facts] ****************************************************************************************************************************
ok: [ubuntu]
ok: [centos7]

TASK [Print OS] ***********************************************************************************************************************************
ok: [centos7] => {
    "msg": "CentOS"
}
ok: [ubuntu] => {
    "msg": "Ubuntu"
}

TASK [Print fact] *********************************************************************************************************************************
ok: [centos7] => {
    "msg": "el default fact"
}
ok: [ubuntu] => {
    "msg": "deb default fact"
}

PLAY RECAP ****************************************************************************************************************************************
centos7                    : ok=3    changed=0    unreachable=0    failed=0    skipped=0    rescued=0    ignored=0   
ubuntu                     : ok=3    changed=0    unreachable=0    failed=0    skipped=0    rescued=0    ignored=0 
```

----
9. *Посмотрите при помощи `ansible-doc` список плагинов для подключения. Выберите подходящий для работы на `control node`.*  
  
```
Anastasia@MacBook-Air playbook % ansible-doc --type=connection -l
[WARNING]: Collection ibm.qradar does not support Ansible version 2.12.6
[WARNING]: Collection splunk.es does not support Ansible version 2.12.6
[DEPRECATION WARNING]: ansible.netcommon.napalm has been deprecated. See the plugin documentation for more details. This feature will be removed 
from ansible.netcommon in a release after 2022-06-01. Deprecation warnings can be disabled by setting deprecation_warnings=False in ansible.cfg.
ansible.netcommon.httpapi      Use httpapi to run command on network appliances                                                               
ansible.netcommon.libssh       (Tech preview) Run tasks using libssh for ssh connection                                                       
ansible.netcommon.napalm       Provides persistent connection using NAPALM                                                                    
ansible.netcommon.netconf      Provides a persistent connection using the netconf protocol                                                    
ansible.netcommon.network_cli  Use network_cli to run command on network appliances                                                           
ansible.netcommon.persistent   Use a persistent unix socket for connection                                                                    
community.aws.aws_ssm          execute via AWS Systems Manager                                                                                
community.docker.docker        Run tasks in docker containers                                                                                 
community.docker.docker_api    Run tasks in docker containers                                                                                 
community.docker.nsenter       execute on host running controller container                                                                   
community.general.chroot       Interact with local chroot                                                                                     
community.general.funcd        Use funcd to connect to target                                                                                 
community.general.iocage       Run tasks in iocage jails                                                                                      
community.general.jail         Run tasks in jails                                                                                             
community.general.lxc          Run tasks in lxc containers via lxc python library                                                             
community.general.lxd          Run tasks in lxc containers via lxc CLI                                                                        
community.general.qubes        Interact with an existing QubesOS AppVM                                                                        
community.general.saltstack    Allow ansible to piggyback on salt minions                                                                     
community.general.zone         Run tasks in a zone instance                                                                                   
community.libvirt.libvirt_lxc  Run tasks in lxc containers via libvirt                                                                        
community.libvirt.libvirt_qemu Run tasks on libvirt/qemu virtual machines                                                                     
community.okd.oc               Execute tasks in pods running on OpenShift                                                                     
community.vmware.vmware_tools  Execute tasks inside a VM via VMware Tools                                                                     
community.zabbix.httpapi       Use httpapi to run command on network appliances                                                               
containers.podman.buildah      Interact with an existing buildah container                                                                    
containers.podman.podman       Interact with an existing podman container                                                                     
kubernetes.core.kubectl        Execute tasks in pods running on Kubernetes                                                                    
local                          execute on controller                                                                                          
paramiko_ssh                   Run tasks via python ssh (paramiko)                                                                            
psrp                           Run tasks over Microsoft PowerShell Remoting Protocol                                                          
ssh                            connect via SSH client binary                                                                                  
winrm                          Run tasks over Microsoft's WinRM   
```

**Выбираем `local`**

----
10. *В `prod.yml` добавьте новую группу хостов с именем  `local`, в ней разместите localhost с необходимым типом подключения.*  

**Добавляем `inventory/prod.yml`:**
```yml
---
  el:
    hosts:
      centos7:
        ansible_connection: docker
  deb:
    hosts:
      ubuntu:
        ansible_connection: docker

  local:
    hosts:
      localhost:
        ansible_connection: local
```

----
11. *Запустите playbook на окружении `prod.yml`. При запуске `ansible` должен запросить у вас пароль. Убедитесь что факты `some_fact` для каждого из хостов определены из верных `group_vars`.*  

```
Anastasia@MacBook-Air playbook % ansible-playbook -i inventory/prod.yml site.yml --ask-vault-pass
Vault password: 

PLAY [Print os facts] *******************************************************************************************************************************************

TASK [Gathering Facts] ******************************************************************************************************************************************
[WARNING]: Platform darwin on host localhost is using the discovered Python interpreter at /opt/homebrew/bin/python3.9, but future installation of another
Python interpreter could change the meaning of that path. See https://docs.ansible.com/ansible-core/2.12/reference_appendices/interpreter_discovery.html for
more information.
ok: [localhost]
ok: [ubuntu]
ok: [centos7]

TASK [Print OS] *************************************************************************************************************************************************
ok: [centos7] => {
    "msg": "CentOS"
}
ok: [ubuntu] => {
    "msg": "Ubuntu"
}
ok: [localhost] => {
    "msg": "MacOSX"
}

TASK [Print fact] ***********************************************************************************************************************************************
ok: [centos7] => {
    "msg": "el default fact"
}
ok: [ubuntu] => {
    "msg": "deb default fact"
}
ok: [localhost] => {
    "msg": "all default fact"
}

PLAY RECAP ******************************************************************************************************************************************************
centos7                    : ok=3    changed=0    unreachable=0    failed=0    skipped=0    rescued=0    ignored=0   
localhost                  : ok=3    changed=0    unreachable=0    failed=0    skipped=0    rescued=0    ignored=0   
ubuntu                     : ok=3    changed=0    unreachable=0    failed=0    skipped=0    rescued=0    ignored=0  
```

----
12. *Заполните `README.md` ответами на вопросы. Сделайте `git push` в ветку `master`. В ответе отправьте ссылку на ваш открытый репозиторий с изменённым `playbook` и заполненным `README.md`.*

**Выполнено! Ссылка на [репозиторий](https://github.com/procenko-anastasia/netology8)**
