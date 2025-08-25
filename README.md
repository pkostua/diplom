# Дипломный практикум в Yandex.Cloud
https://github.com/netology-code/devops-diplom-yandexcloud

---

## Создание облачной инфраструктуры
На этом этапе у нас есть яндекс облако, из кторого мы достали folder_id, cloud_id и OAuth token.  
| Секрет           | Значение                                                                 |
|------------------|--------------------------------------------------------------------------|
| folder_id        |  https://yandex.cloud/ru/docs/resource-manager/operations/folder/get-id  |
| cloud_id         | https://yandex.cloud/ru/docs/resource-manager/operations/cloud/get-id    |
| token            | https://yandex.cloud/ru/docs/iam/concepts/authorization/oauth-token      |

С помощью проекта bootstrap https://github.com/pkostua/diplom-bootstrap создаем s3 хранилище для terraform state и сервисный аккаунт для terraform.

В результате работы проекта bootstrap мы получили доступы к s3 бакету. Также в yandex cloud console мы может получит файл для авторизации сервисного аккаунта sa_kay.json.
Это пригодится нам на следующих этапах.

---

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

---

## Создание тестового приложения
Репозиторий приложения. https://github.com/pkostua/diplom-test-app  
Для работы автоматической сборки и деплоя требуется добавить вот такие сикреты  

- `REGISTRY_URL` - URL Docker registry (например: cr.yandex\{registry_id})  
- `YC_OAUTH_TOKEN` — OAuth-токен сервисного аккаунта в Yandex Cloud
- `YC_CLOUD_ID` — ID облака
- `YC_FOLDER_ID` — ID каталога
- `YC_CLUSTER_ID` — ID кластера Managed Kubernetes

### При комите происходит только сборка и отправка в реестр
<img width="1485" height="662" alt="image" src="https://github.com/user-attachments/assets/68848f9a-25a4-4de8-b25d-06d2444cff84" />

### При создании тега - сборка и деплой в кластер
<img width="1555" height="653" alt="image" src="https://github.com/user-attachments/assets/cf76d9bb-f2f9-4550-8306-4f8a744495f7" />

---

## Подготовка cистемы мониторинга и деплой приложения
Проект, кторый пороизводит установку мониторинга и приложения https://github.com/pkostua/diplom-k8s  
Систему мониторинга я решил делать свою, потомучто  kube-prometheus меня встрелил предупреждением, о том, что это все временно и может не работать, а в bitnami не нашелся alertmanager.
Набор манифестов деплоит Prometheus, Alertmanager, node-exporter, kube-state-metrics, Grafana, добавляет ingress nginx, вывешивает интерфейс grafana через ingress + балансировщик  на http://grafana.pkdp.ru

```
логин: admin
пароль: MSPrd123!
```
А также деплоит приложение test-app и вывешивает его через балансировщик на http://test-app.pkdp.ru

Для запуска нужно
1. Установить nginx ingress `install-nginx-ingress-simple.sh`
2. Задеплоить моиторинг и приложение `deploy.sh`

Для удаления можно воспользоваться `cleanup.sh`

### Поды мониторинга
<img width="2360" height="558" alt="image" src="https://github.com/user-attachments/assets/bb791281-9f73-4fdc-a6f6-6606b11b9254" />

### Grafana dashboard
<img width="2565" height="1168" alt="image" src="https://github.com/user-attachments/assets/a1130134-bcb2-4d1c-84d7-d57394fd8a55" />


---
## Завершение
Просим владельца домена прописать ip адрес балансировщика для поддоменов test-app grafana.
Сейчас приложение доступно по ссылке  
http://test-app.pkdp.ru  
http://grafana.pkdp.ru

---
## Итоги

В рамках этой работы мы сделали следующее

- Подготовили статическую инфраструктуру для хранения  состояния и исполнения динамической инфраструктуры
- Подготовили динамическую инфраструктуру и ее непрерывную доставку
- Подготовили кластер k8s с мониторингом
- Подготовили непрерывную сборку доставку приложения в кластер






