# Домашнее задание к занятию "08.01 Введение в Ansible"

## Подготовка к выполнению
1. Установите ansible версии 2.10 или выше.
2. Создайте свой собственный публичный репозиторий на github с произвольным именем.
3. Скачайте [playbook](./playbook/) из репозитория с домашним заданием и перенесите его в свой репозиторий.

## Основная часть
1. Попробуйте запустить playbook на окружении из `test.yml`, зафиксируйте какое значение имеет факт `some_fact` для указанного хоста при выполнении playbook'a.
```
centos@node01:~/mnt-hw/08-ansible-01-base/playbook$ ansible-playbook -i inventory/test.yml site.yml
...
TASK [Print fact] ****************************************************************************************
ok: [localhost] => {
    "msg": 12
}
```
2. Найдите файл с переменными (group_vars) в котором задаётся найденное в первом пункте значение и поменяйте его на 'all default fact'.
```
centos@node01:~/mnt-hw/08-ansible-01-base/playbook$ cat group_vars/all/examp.yml
---
  some_fact: all default fact
```
centos@node01:~/mnt-hw/08-ansible-01-base/playbook$ ansible-playbook -i inventory/test.yml site.yml
```
...
TASK [Print fact] *****************************************************************************************
ok: [localhost] => {
    "msg": "all default fact"
}
```
3. Воспользуйтесь подготовленным (используется `docker`) или создайте собственное окружение для проведения дальнейших испытаний.
4. Проведите запуск playbook на окружении из `prod.yml`. Зафиксируйте полученные значения `some_fact` для каждого из `managed host`.
```
$ ansible-playbook -i inventory/prod.yml site.yml
...
TASK [Print fact] *****************************************************************************************
ok: [centos7] => {
    "msg": "el"
}
ok: [ubuntu] => {
    "msg": "deb"
}

```
5. Добавьте факты в `group_vars` каждой из групп хостов так, чтобы для `some_fact` получились следующие значения: для `deb` - 'deb default fact', для `el` - 'el default fact'.
6. Повторите запуск playbook на окружении `prod.yml`. Убедитесь, что выдаются корректные значения для всех хостов.
```
$ ansible-playbook -i inventory/prod.yml site.yml
...
TASK [Print fact] *****************************************************************************************
ok: [centos7] => {
    "msg": "el default fact"
}
ok: [ubuntu] => {
    "msg": "deb default fact"
}

```
7. При помощи `ansible-vault` зашифруйте факты в `group_vars/deb` и `group_vars/el` с паролем `netology`.
```
$ ansible-vault encrypt group_vars/deb/examp.yml
New Vault password:
Confirm New Vault password:
Encryption successful

$ ansible-vault encrypt group_vars/el/examp.yml
New Vault password:
Confirm New Vault password:
Encryption successful
```
8. Запустите playbook на окружении `prod.yml`. При запуске `ansible` должен запросить у вас пароль. Убедитесь в работоспособности.
```
$ ansible-playbook -i inventory/prod.yml site.yml --ask-vault-pass
Vault password:

PLAY [Print os facts] ******************************************************************************************
...
PLAY RECAP *****************************************************************************************************
centos7                    : ok=3    changed=0    unreachable=0    failed=0    skipped=0    rescued=0    ignored=0
ubuntu                     : ok=3    changed=0    unreachable=0    failed=0    skipped=0    rescued=0    ignored=0

```
9. Посмотрите при помощи `ansible-doc` список плагинов для подключения. Выберите подходящий для работы на `control node`.
```
Не совсем понял вопрос. Если речь о нодах запущенных в тестовом задании, то плагин "community.docker"
```

10. В `prod.yml` добавьте новую группу хостов с именем  `local`, в ней разместите localhost с необходимым типом подключения.
```
$ cat inventory/prod.yml
---
...
  local:
    hosts:
      localhost:
        ansible_connection: local

```
11. Запустите playbook на окружении `prod.yml`. При запуске `ansible` должен запросить у вас пароль. Убедитесь что факты `some_fact` для каждого из хостов определены из верных `group_vars`.
```
...
TASK [Print fact] ************************************************************************************************
ok: [localhost] => {
    "msg": "all default fact"
}
ok: [centos7] => {
    "msg": "el default fact"
}
ok: [ubuntu] => {
    "msg": "deb default fact"
}

```
12. Заполните `README.md` ответами на вопросы. Сделайте `git push` в ветку `master`. В ответе отправьте ссылку на ваш открытый репозиторий с изменённым `playbook` и заполненным `README.md`.

## Необязательная часть

1. При помощи `ansible-vault` расшифруйте все зашифрованные файлы с переменными.
```
$ ansible-vault decrypt group_vars/deb/examp.yml
$ ansible-vault decrypt group_vars/el/examp.yml

```
3. Зашифруйте отдельное значение `PaSSw0rd` для переменной `some_fact` паролем `netology`. Добавьте полученное значение в `group_vars/all/exmp.yml`.
```
$ ansible-vault encrypt_string PaSSw0rd
!vault |
          $ANSIBLE_VAULT;1.1;AES256
          39653230356362363864353763313034636463326662333838323434666435333463613131333262
          3033633061653638373036343063653661363639363238350a313837646533383261643366386638
          66663831663766626361653035656361373531646534323135333563643636346636663334613264
          6137343464383436640a623837313830333434346335393330383961613962356166346462633865
          3438

```
4. Запустите `playbook`, убедитесь, что для нужных хостов применился новый `fact`.
```
$ ansible-playbook -i inventory/prod.yml site.yml --ask-vault-pass
...
TASK [Print fact] ************************************************************************************
ok: [localhost] => {
    "msg": "PaSSw0rd"
}
ok: [centos7] => {
    "msg": "el default fact"
}
ok: [ubuntu] => {
    "msg": "deb default fact"
}

```
6. Добавьте новую группу хостов `fedora`, самостоятельно придумайте для неё переменную. В качестве образа можно использовать [этот](https://hub.docker.com/r/pycontribs/fedora).
7. Напишите скрипт на bash: автоматизируйте поднятие необходимых контейнеров, запуск ansible-playbook и остановку контейнеров.
8. Все изменения должны быть зафиксированы и отправлены в вашей личный репозиторий.

---

### Как оформить ДЗ?

Выполненное домашнее задание пришлите ссылкой на .md-файл в вашем репозитории.

---
