## Clickhouse-Vector-Lighthouse Ansible-Playbook
Проект развёртывания инфраструктуры для сбора и анализа логов на основе Clickhouse и Vector и просмотра в Lighthouse.

В плейбуке имеется 3 PLAY для настройки, соответственно, Clickhouse в качестве БД, Vector в качестве транспорта и Lighthouse в качестве просмотрщика таблицы с логами. В конце исполнения плейбука выдаётся адрес для подключения к Lighthouse.

## Installation
Вам необходимо создать 3 виртуальных машины.

Для установки использовать дистрибутив CentOS 7.

Для выбора версии и архитектуры замените в файлах `group_vars/clickhouse/clickhouse.yml` и `group_vars/vector/vector.yml` соответствующие значения переменных:
```
clickhouse_version: <"version">
clickhouse_arch: <"arch">
```
```
vector_version: <"version">
vector_arch: <"arch">
```

При необходимости можно изменить пользователей, от имени которых подключается Ansible, в параметре `ansible_user` в файле `/inventory/prod.yml`

### Install
Для запуска можно использовать следующую команду после того, как хосты были прописаны в `inventory/prod.yml`:
```shell
ansible-playbook -i inventory/prod.yml site.yml
```


