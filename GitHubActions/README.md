### Создание ресурсов Terraform через CI/CD Github Actions

---

#### Шаг 1. 

- Создание бакета для 01-bootstrap Terraform state (один раз)
- Создание service account для CI/CD (один раз)

```bash
# Создание бакета для 01-bootstrap
yc storage bucket create --name diplom-01bootstrap-tfstate

# Создаём SA
yc iam service-account create --name ci-cd-sa

# Даём права editor на каталог
yc resource-manager folder add-access-binding <folder_id> \
  --service-account-name ci-cd-sa \
  --role admin

# JSON-ключ для Terraform provider
yc iam key create --service-account-name ci-cd-sa -o ci-cd-sa.json

# Статический ключ для S3-бэкенда (Object Storage)
yc iam access-key create --service-account-name ci-cd-sa
# Выведет AccessKeyId (YCAJE...) и Secret (YCN...) — это AWS_ACCESS_KEY_ID / AWS_SECRET_ACCESS_KEY
```

![diplom_git1-01](https://github.com/IthnHuitn/diplom/blob/master/scr/diplom_git1-01.png)
---


#### Шаг 2. Добавление секретов в GitHub Secrets для репозитория с terraform

```bash
# Доступ к Yandex Object Storage (S3 backend)
gh secret set AWS_ACCESS_KEY_ID --body "YCAJExxxxx"
gh secret set AWS_SECRET_ACCESS_KEY --body "YCNxxxxx"

# Доступ к Yandex Cloud
gh secret set YC_CLOUD_ID --body "<cloud_id>"
gh secret set YC_FOLDER_ID --body "<folder_id>"

# JSON-ключ сервисного аккаунта
gh secret set YC_SA_KEY < ci-cd-sa.json

# SSH-ключ для доступа к нодам (для Terraform provisioners)
gh secret set SSH_PRIVATE_KEY < ~/.ssh/"<ssh-key>"
```
![diplom_git1-02](https://github.com/IthnHuitn/diplom/blob/master/scr/diplom_git1-02.png)

---


#### Шаг 3. Загрузка файлов в репозиторий и старт сборки кластера

![diplom_git1-03](https://github.com/IthnHuitn/diplom/blob/master/scr/diplom_git1-03.png)
---
![diplom_git1-04](https://github.com/IthnHuitn/diplom/blob/master/scr/diplom_git1-04.png)
---
**Terraform Output**
![diplom_git1-05](https://github.com/IthnHuitn/diplom/blob/master/scr/diplom_git1-05.png)
---
**tfstate всех слоёв в YC Registry**
![diplom_git1-07](https://github.com/IthnHuitn/diplom/blob/master/scr/diplom_git1-07.png)
---
![diplom_git1-08](https://github.com/IthnHuitn/diplom/blob/master/scr/diplom_git1-08.png)

##### Копирование файла настроек на локальную машину для доступа к кластеру
```bash
ssh -i ~/.ssh/vm5 debian@<master_public_ip> 'sudo cat /etc/kubernetes/admin.conf' | sed 's/<master_internal_ip>/<master_public_ip>/g' > ~/.kube/config
chmod 600 ~/.kube/config
```
##### Доступ к ресурсам
```bash
kubectl get pods --all-namespaces
```
![diplom_git1-06](https://github.com/IthnHuitn/diplom/blob/master/scr/diplom_git1-06.png)

#### Шаг 4. Деплой приложения

##### Обновляю секреты в репозиторий тестового приложения через GitHub Secrets
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
![diplom_git1-09](https://github.com/IthnHuitn/diplom/blob/master/scr/diplom_git1-09.png)


##### Записываю тег для приложения
```bash
git tag v.1.0.0
git push origin v1.0.0
```
![diplom_git1-10](https://github.com/IthnHuitn/diplom/blob/master/scr/diplom_git1-10.png)
---
![diplom_git1-11](https://github.com/IthnHuitn/diplom/blob/master/scr/diplom_git1-11.png)
---
![diplom_git1-12](https://github.com/IthnHuitn/diplom/blob/master/scr/diplom_git1-12.png)
---
Страница приложения
![diplom_git1-13](https://github.com/IthnHuitn/diplom/blob/master/scr/diplom_git1-13.png)
---

**Сворачивание инфраструктуры реализовано через ручной запуск workflow.**
![diplom_git1-14](https://github.com/IthnHuitn/diplom/blob/master/scr/diplom_git1-14.png)
---
![diplom_git1-15](https://github.com/IthnHuitn/diplom/blob/master/scr/diplom_git1-15.png)
---
![diplom_git1-16](https://github.com/IthnHuitn/diplom/blob/master/scr/diplom_git1-16.png)

---
