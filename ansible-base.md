## Домашнее задание к занятию "08.01 Введение в Ansible"  
### Подготовка к выполнению 
1. Установите ansible версии 2.10 или выше.
```
root@anton-v-m:~/netology# ansible --version
ansible [core 2.12.6]
  config file = /etc/ansible/ansible.cfg
  configured module search path = ['/root/.ansible/plugins/modules', '/usr/share/ansible/plugins/modules']
  ansible python module location = /root/.local/lib/python3.8/site-packages/ansible
  ansible collection location = /root/.ansible/collections:/usr/share/ansible/collections
  executable location = /usr/bin/ansible
  python version = 3.8.10 (default, Mar 15 2022, 12:22:08) [GCC 9.4.0]
  jinja version = 3.1.2
  libyaml = True

```
2. Создайте свой собственный публичный репозиторий на github с произвольным именем.

https://github.com/AntonKonyakhin/ansible-base.git

3. Скачайте playbook из репозитория с домашним заданием и перенесите его в свой репозиторий.

## Основная часть
1. Попробуйте запустить playbook на окружении из `test.yml`, зафиксируйте какое значение имеет факт `some_fact` для указанного хоста при выполнении playbook'a.  
```
root@anton-v-m:~/netology/git-playbook# ansible-playbook site.yml -i inventory/test.yml 

PLAY [Print os facts] ******************************************************************************************************************

TASK [Gathering Facts] *****************************************************************************************************************
ok: [localhost]

TASK [Print OS] ************************************************************************************************************************
ok: [localhost] => {
    "msg": "Ubuntu"
}

TASK [Print fact] **********************************************************************************************************************
ok: [localhost] => {
    "msg": 12
}

PLAY RECAP *****************************************************************************************************************************
localhost                  : ok=3    changed=0    unreachable=0    failed=0    skipped=0    rescued=0    ignored=0 
```

some_fact - "msg": 12

2. Найдите файл с переменными (group_vars) в котором задаётся найденное в первом пункте значение и поменяйте его на 'all default fact'.

Ответ: поменял значение в файле  group_vars/all/examp.yml  
получилось так:  
```
root@anton-v-m:~/netology/git-playbook# ansible-playbook site.yml -i inventory/test.yml 

PLAY [Print os facts] ******************************************************************************************************************

TASK [Gathering Facts] *****************************************************************************************************************
ok: [localhost]

TASK [Print OS] ************************************************************************************************************************
ok: [localhost] => {
    "msg": "Ubuntu"
}

TASK [Print fact] **********************************************************************************************************************
ok: [localhost] => {
    "msg": "all default fact"
}

PLAY RECAP *****************************************************************************************************************************
localhost                  : ok=3    changed=0    unreachable=0    failed=0    skipped=0    rescued=0    ignored=0   

```

3. Воспользуйтесь подготовленным (используется docker) или создайте собственное окружение для проведения дальнейших испытаний.  
запустил
```
docker pull pycontribs/centos:7
docker pull pycontribs/ubuntu:latest
```
```
root@anton-v-m:~/netology/git-playbook# docker run --name ubuntu -d pycontribs/ubuntu sleep 6000000
3e065fdf4c68f127be3339f637136aea717a9ca08d0bcb8e6d2c6b944660cedd
root@anton-v-m:~/netology/git-playbook# docker run --name centos7 -d pycontribs/centos:7 sleep 6000000
d47ab0d5449564fbe312fb523e8008c692b3610f92d69886618ec8336539050a


```
```
root@anton-v-m:~/netology/git-playbook# docker ps
CONTAINER ID   IMAGE                 COMMAND           CREATED          STATUS          PORTS     NAMES
d47ab0d54495   pycontribs/centos:7   "sleep 6000000"   3 seconds ago    Up 3 seconds              centos7
3e065fdf4c68   pycontribs/ubuntu     "sleep 6000000"   23 seconds ago   Up 22 seconds             ubuntu

```
4. Проведите запуск playbook на окружении из `prod.yml`. Зафиксируйте полученные значения `some_fact` для каждого из managed host.  

```
root@anton-v-m:~/netology/git-playbook# ansible-playbook site.yml -i inventory/prod.yml 

PLAY [Print os facts] ******************************************************************************************************************

TASK [Gathering Facts] *****************************************************************************************************************
ok: [ubuntu]
ok: [centos7]

TASK [Print OS] ************************************************************************************************************************
ok: [centos7] => {
    "msg": "CentOS"
}
ok: [ubuntu] => {
    "msg": "Ubuntu"
}

TASK [Print fact] **********************************************************************************************************************
ok: [centos7] => {
    "msg": "el"
}
ok: [ubuntu] => {
    "msg": "deb"
}

PLAY RECAP *****************************************************************************************************************************
centos7                    : ok=3    changed=0    unreachable=0    failed=0    skipped=0    rescued=0    ignored=0   
ubuntu                     : ok=3    changed=0    unreachable=0    failed=0    skipped=0    rescued=0    ignored=0 

```
some_fact для centos7    "msg": "el"  
some_fact для ubuntu      "msg": "deb"

5. Добавьте факты в group_vars каждой из групп хостов так, чтобы для some_fact получились следующие значения: для deb - 'deb default fact', для el - 'el default fact'.

поменя значения в файлах:
```
root@anton-v-m:~/netology/git-playbook# vim group_vars/deb/examp.yml 
root@anton-v-m:~/netology/git-playbook# vim group_vars/el/examp.yml 

```
6. Повторите запуск playbook на окружении prod.yml. Убедитесь, что выдаются корректные значения для всех хостов.  

теперь получилось так:
```
root@anton-v-m:~/netology/git-playbook# ansible-playbook site.yml -i inventory/prod.yml 

PLAY [Print os facts] ******************************************************************************************************************

TASK [Gathering Facts] *****************************************************************************************************************
ok: [ubuntu]
ok: [centos7]

TASK [Print OS] ************************************************************************************************************************
ok: [centos7] => {
    "msg": "CentOS"
}
ok: [ubuntu] => {
    "msg": "Ubuntu"
}

TASK [Print fact] **********************************************************************************************************************
ok: [centos7] => {
    "msg": "el default fact"
}
ok: [ubuntu] => {
    "msg": "deb default fact"
}

PLAY RECAP *****************************************************************************************************************************
centos7                    : ok=3    changed=0    unreachable=0    failed=0    skipped=0    rescued=0    ignored=0   
ubuntu                     : ok=3    changed=0    unreachable=0    failed=0    skipped=0    rescued=0    ignored=0   
```

7. При помощи ansible-vault зашифруйте факты в group_vars/deb и group_vars/el с паролем netology.  

```
root@anton-v-m:~/netology/git-playbook# ansible-vault encrypt group_vars/el/examp.yml 
New Vault password: 
Confirm New Vault password: 
Encryption successful
root@anton-v-m:~/netology/git-playbook# ansible-vault encrypt group_vars/deb/examp.yml 
New Vault password: 
Confirm New Vault password: 
```
8. Запустите playbook на окружении prod.yml. При запуске ansible должен запросить у вас пароль. Убедитесь в работоспособности.  
```
root@anton-v-m:~/netology/git-playbook# ansible-playbook site.yml -i inventory/prod.yml --ask-vault-password
Vault password: 

PLAY [Print os facts] ******************************************************************************************************************

TASK [Gathering Facts] *****************************************************************************************************************
ok: [ubuntu]
ok: [centos7]

TASK [Print OS] ************************************************************************************************************************
ok: [centos7] => {
    "msg": "CentOS"
}
ok: [ubuntu] => {
    "msg": "Ubuntu"
}

TASK [Print fact] **********************************************************************************************************************
ok: [centos7] => {
    "msg": "el default fact"
}
ok: [ubuntu] => {
    "msg": "deb default fact"
}

PLAY RECAP *****************************************************************************************************************************
centos7                    : ok=3    changed=0    unreachable=0    failed=0    skipped=0    rescued=0    ignored=0   
ubuntu                     : ok=3    changed=0    unreachable=0    failed=0    skipped=0    rescued=0    ignored=0   

```

9. Посмотрите при помощи ansible-doc список плагинов для подключения. Выберите подходящий для работы на control node.  

```
root@anton-v-m:~/netology/git-playbook# ansible-doc -t connection -l
```

local                          execute on controller 

10. В `prod.yml` добавьте новую группу хостов с именем `local`, в ней разместите `localhost` с необходимым типом подключения.  
```yml
  local:
    hosts:
      localhost:
        ansible_connection: local
```

11. Запустите `playbook` на окружении `prod.yml`. При запуске `ansible` должен запросить у вас пароль. Убедитесь что факты `some_fact` для каждого из хостов определены из верных `group_vars`.  

```
root@anton-v-m:~/netology/git-playbook# ansible-playbook site.yml -i inventory/prod.yml --ask-vault-password
Vault password: 

PLAY [Print os facts] ******************************************************************************************************************

TASK [Gathering Facts] *****************************************************************************************************************
ok: [localhost]
ok: [ubuntu]
ok: [centos7]

TASK [Print OS] ************************************************************************************************************************
ok: [centos7] => {
    "msg": "CentOS"
}
ok: [ubuntu] => {
    "msg": "Ubuntu"
}
ok: [localhost] => {
    "msg": "Ubuntu"
}

TASK [Print fact] **********************************************************************************************************************
ok: [centos7] => {
    "msg": "el default fact"
}
ok: [ubuntu] => {
    "msg": "deb default fact"
}
ok: [localhost] => {
    "msg": "all default fact"
}

PLAY RECAP *****************************************************************************************************************************
centos7                    : ok=3    changed=0    unreachable=0    failed=0    skipped=0    rescued=0    ignored=0   
localhost                  : ok=3    changed=0    unreachable=0    failed=0    skipped=0    rescued=0    ignored=0   
ubuntu                     : ok=3    changed=0    unreachable=0    failed=0    skipped=0    rescued=0    ignored=0   

```

12. Заполните README.md ответами на вопросы. Сделайте git push в ветку master. В ответе отправьте ссылку на ваш открытый репозиторий с изменённым playbook и заполненным README.md. 

### Самоконтроль выполненения задания

1. Где расположен файл с some_fact из второго пункта задания?

group_vars/all/examp.yml

2. Какая команда нужна для запуска вашего playbook на окружении test.yml?

ansible-playbook site.yml -i inventory/test.yml

3. Какой командой можно зашифровать файл?  

ansible-vault encrypt group_vars/deb/examp.yml


4. Какой командой можно расшифровать файл?

ansible-vault decrypt group_vars/deb/examp.yml

5. Можно ли посмотреть содержимое зашифрованного файла без команды расшифровки файла? Если можно, то как?  
```
root@anton-v-m:~/netology/git-playbook# ansible-vault view group_vars/deb/examp.yml
Vault password: 

---
  some_fact: "deb default fact"
```

6. Как выглядит команда запуска playbook, если переменные зашифрованы?

ansible-playbook site.yml -i inventory/prod.yml --ask-vault-password

7. Как называется модуль подключения к host на windows?

winrm                          Run tasks over Microsoft's WinRM

8. Приведите полный текст команды для поиска информации в документации ansible для модуля подключений ssh

root@anton-v-m:~/netology/git-playbook# ansible-doc -t connection ssh


9. Какой параметр из модуля подключения ssh необходим для того, чтобы определить пользователя, под которым необходимо совершать подключение?


- remote_user  
        User name with which to login to the remote server, normally set by the remote_user keyword.
        If no user is supplied, Ansible will let the SSH client binary choose the user as it normally.
        [Default: (null)]
