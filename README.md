# Дипломный практикум в Yandex.Cloud
https://github.com/netology-code/devops-diplom-yandexcloud

## Создание облачной инфраструктуры
На этом этапе у нас есть яндекс облако, из кторого мы достали folder_id, cloud_id и OAuth token.  
С помощью проекта bootstrap https://github.com/pkostua/diplom-bootstrap создаем s3 хранилище для terraform state и сервисный аккаунт для terraform.

В результате работы проекта bootstrap мы получили доступы к s3 бакету. Также в yandex cloud console мы может получит файл для авторизации сервисного аккаунта sa_kay.json.
Это пригодится нам на следующих этапах.

## Создание Kubernetes кластера + CI для terraform
На этом этапе нужно создать кластер k8s. Я выбрал вариант Managed Service for Kubernetes потомучто мне он видится более отказоустойчивым за те же деньги.  
Проект https://github.com/pkostua/diplom-infrastructure создает кластер и репозиторий для хранения образов.
Для автоматического применения изменений необходимо в сикреты github actions добавить данные, полученный при создании инфраструктуры
| Секрет           | Значение                                                          |
|------------------|-------------------------------------------------------------------|
| `YC_CLOUD_ID`    | `xxxxxxxxxxxxxxxxxxxx`                                            |
| `YC_FOLDER_ID`   | `xxxxxxxxxxxxxxxxx`                                               |
| `YC_SA_KEY`      | Содержимое файла `sa-key.json`                                    |
| `SSH_PUBLIC_KEY` | Публичный ключ для доступа к нодам (должен быть у каждого девопса)|

В случае успешного применения запусится и отработает action
### github action
<img width="1452" height="935" alt="image" src="https://github.com/user-attachments/assets/24a937d5-07b5-431e-83d9-e0d601232abf" />  
### Последние строки лога
<img width="1147" height="935" alt="image" src="https://github.com/user-attachments/assets/909446c7-ae6b-4f34-aa82-8d62074fdbb3" />  
* Примечание: время там показывают неверное. На скриншоте перезапуск изза того что на воркер ноды не хватило ресурсов. Реальное время выполенения около 20 минут.  

### Результат
<img width="1483" height="753" alt="image" src="https://github.com/user-attachments/assets/f0c086b2-ed1d-485a-b15e-e6d00f120837" />  

На этом этапе мы можем получить файл авторизции для управления кластером

```bash
     yc managed-kubernetes cluster get-credentials --id cata3me84qd2p9kj3v74 --external
```
В полученном конфиге нужно добавить в URL порт api k8s : 6443 

```
    server: https://51.250.93.189:6443
```

## Создание тестового приложения


## Подготовка cистемы мониторинга и деплой приложения

Систему мониторинга я решил делать свою. потомучто  kube-prometheus мне встрелил предупреждением, о том, что это все временно и может не работать, а в bitnami не нашелся alertmanager
Набор манифестов деплоит Prometheus, Alertmanager, node-exporter, kube-state-metrics, Grafana. добавляет ingress nginx, вывешивает интерфейс grafana через ingress + балансировщик  на 

```
http://grafana.pkdp.ru
логин: admin
пароль: MSPrd123!
```

Для запуска нужно
1. Установить nginx ingress install-nginx-ingress-simple.sh
2. Задеплоить моиторинг и приложение deploy.sh

Для удаления можно воспользоваться cleanup.sh






