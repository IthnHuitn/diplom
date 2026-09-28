### Создание ресурсов с локального терминала

#### Шаг 1. Создание первого слоя с секретами, сервисными аккаунтами и YC Registry

```bash
/01-bootstrap$ terraform init
/01-bootstrap$ terraform plan
/01-bootstrap$ terraform apply
```

##### Один раз за сессию добавляю секреты в локальное  окружение:
```bash
source /tmp/.env # файл создаётся при создании первого слоя 
```
##### Добавляю ssh-ключ в агента для доступа мастера к рабочим нодам
```bash
ssh-add ~/.ssh/"<ssh-key>"
# проверить наличие
ssh-add -l
# права для ключа
chmod 600 ~/.ssh/"<ssh-key>"
```

##### Шаг 2. Создание остальных слоёв инфраструктуры

```bash
/02-network$ terraform init
/02-network$ terraform plan
/02-network$ terraform apply
```

```bash
/03-infrastructure$ terraform init
/03-infrastructure$ terraform plan
/03-infrastructure$ terraform apply
```

#### Вывод всех ресурсов:
```bash
for dir in 01-bootstrap 02-network 03-infrastructure; do echo "=== $dir ===" && cd "$dir" && terraform output && cd ..; done
```
![diplom_loc1-02.png](https://github.com/IthnHuitn/diplom/blob/master/scr/diplom_loc1-02.png)
---
![diplom_loc1-03.png](https://github.com/IthnHuitn/diplom/blob/master/scr/diplom_loc1-03.png)

##### Ресурсы K8S
```bash
kubectl get pods --all-namespaces
kubectl get pods -n kube-system
```
![diplom_loc1-04.png](https://github.com/IthnHuitn/diplom/blob/master/scr/diplom_loc1-04.png)

##### Доступ к Grafana

Первичный вход со стандартными значениями:
```
login    = admin
password = admin
```
![diplom_loc1-06.png](https://github.com/IthnHuitn/diplom/blob/master/scr/diplom_loc1-06.png)
---
![diplom_loc1-05.png](https://github.com/IthnHuitn/diplom/blob/master/scr/diplom_loc1-05.png)
---

##### Шаг 3. Деплой приложения из репозитория с тегом 1.0.0

##### Добавляю секреты в репозиторий тестового приложения через GitHub Secrets
```bash
# YC_REGISTRY_ID — просто строка
gh secret set YC_REGISTRY_ID --body "<твой_registry_id>"
# KUBECONFIG — содержимое файла в кодировке base64
base64 ~/.kube/config | gh secret set KUBECONFIG
# YC_SA_KEY — JSON-ключ сервисного аккаунта
gh secret set YC_SA_KEY < /home/evilmc/.authorized_key_diplom.json
# LB_PUBLIC_IP — актуальный IP-адрес loadbalancer'a
gh secret set LB_PUBLIC_IP --body "<твой_LB_IP>" 
```
![diplom_loc1-07.png](https://github.com/IthnHuitn/diplom/blob/master/scr/diplom_loc1-07.png)
---

##### Записываю новый тег в репозиторий
![diplom_loc1-08.png](https://github.com/IthnHuitn/diplom/blob/master/scr/diplom_loc1-08.png)
---

##### Запуск и деплой приложения
![diplom_loc1-09.png](https://github.com/IthnHuitn/diplom/blob/master/scr/diplom_loc1-09.png)
---
![diplom_loc1-10.png](https://github.com/IthnHuitn/diplom/blob/master/scr/diplom_loc1-10.png)
---

##### Страница приложения
![diplom_loc1-12.png](https://github.com/IthnHuitn/diplom/blob/master/scr/diplom_loc1-12.png)
---
##### Ресурсы приложения в K8S
![diplom_loc1-13.png](https://github.com/IthnHuitn/diplom/blob/master/scr/diplom_loc1-13.png)
---
##### Образ с тегом в реестре
![diplom_loc1-14.png](https://github.com/IthnHuitn/diplom/blob/master/scr/diplom_loc1-14.png)

---