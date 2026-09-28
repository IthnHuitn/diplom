# Домашнее задание к занятию "`Дипломный практикум в Yandex.Cloud`" - `Ефимов Вячеслав`
 
---

### Описание проекта

Проект разворачивает полностью рабочую инфраструктуру в Yandex Cloud: 
- виртуальные машины
- сеть
- security groups
- Kubernetes-кластер 
- Container Registry. 

Инфраструктура описана в Terraform и разбита на три слоя:
- **01-bootstrap** — сервисные аккаунты, ключи и YC Registry
- **02-network** — VPC, подсети, security groups
- **03-infrastructure** — compute-инстансы и кластер Kubernetes (1 master + 3 worker-ноды)
---
- **Terraform CI/CD** — автоматический запуск `terraform plan` и `terraform apply` при коммите в main

На кластер устанавливаются:
- **ingress-nginx** — Ingress-контроллер для маршрутизации трафика
- **kube-prometheus-stack** — мониторинг через Prometheus, Grafana и Alertmanager (развёртывается через Ansible)
- приложение `test-app` собирается в Docker-образ и пушится в YC Registry через GitHub Actions CI/CD.

`Приложение доступно по публичному IP loadbalancer'а, Grafana — по тому же IP на порту 80 через Ingress.`

### Ссылки репозиториев с кодом Terraform, Ansible Playbook и Тестовым Приложением

[Terraform K8S Cluster+Monitoring](https://github.com/IthnHuitn/k8s_cluster-monitoring) | [Ansible Playbook](https://github.com/IthnHuitn/ansible-k8s) | [Test-App](https://github.com/IthnHuitn/test-app)

---

[Создание ресурсов Terraform через CI/CD Github Actions](https://github.com/IthnHuitn/diplom/blob/master/GitHubActions/README.md)

---

[Создание ресурсов с локального терминала](https://github.com/IthnHuitn/diplom/blob/master/local/README.md)

---