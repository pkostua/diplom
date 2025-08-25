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
