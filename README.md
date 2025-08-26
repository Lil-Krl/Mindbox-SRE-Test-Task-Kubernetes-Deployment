# Mindbox-SRE-Test-Task-Kubernetes-Deployment
## Описание

Это тестовое задание на позицию SRE в Mindbox.  
Проект демонстрирует развертывание отказоустойчивого веб-приложения в Kubernetes с учётом:

- мультизонального кластера (3 зоны)
- дневного цикла нагрузки
- минимального потребления ресурсов

---

## Структура проекта
- `namespace.yml` — создание отдельного namespace `production` для изоляции ресурсов.
- `deployment-service.yml` — Deployment с 4 подами, `topologySpreadConstraints`, `readinessProbe` и `livenessProbe`, а также Service типа `LoadBalancer`.
- `hpa.yml` — Horizontal Pod Autoscaler для автоматического масштабирования подов по CPU от 2 до 6 реплик.

---

## Особенности реализации

1. **Deployment**
   - 4 пода для пикового трафика.
   - Используется `topologySpreadConstraints` для распределения подов по трём зонам.
   - Контейнеру выделены минимальные ресурсы:
     - CPU: 100m на старт, до 1 CPU лимит.
     - Memory: 128Mi на старт, до 256Mi лимит.
   - `readinessProbe` и `livenessProbe` следят за состоянием приложения.

2. **Service**
   - Тип `LoadBalancer` позволяет получить внешний IP для доступа к приложению.
   - Селектор `app: my-app` направляет трафик только на готовые поды.

3. **HPA**
   - Минимум 2 пода ночью для экономии ресурсов.
   - Максимум 6 подов при пиковом трафике.
   - Масштабирование по CPU (средняя загрузка 50%).

---

## Развёртывание

1. **Клонируем репозиторий**

```bash
git clone https://github.com/Lil-Krl/Mindbox-SRE-Test-Task-Kubernetes-Deployment.git
cd Mindbox-SRE-Test-Task-Kubernetes-Deployment
```
2. **Применяем namespace**

```bash
kubectl apply -f namespace.yml
```
3. **Применяем Deployment и Service**

```bash
kubectl apply -f deployment-service.yml
```
4. **Применяем HPA**

```bash
kubectl apply -f hpa.yml
```

---

## Проверка работы

1. **Проверяем поды**
```bash
kubectl get pods -n production
```
Все поды должны быть в статусе `Running` и проходить `READY` проверки.

2. **Проверяем сервис**
```bash
kubectl get svc -n production
```
Сервис должен иметь внешний IP. Проверяем отклик через браузер или `curl`.

3. **Проверяем HPA**
```bash
kubectl get hpa -n production
```
Проверяем, что количество реплик соответствует заданным условиям (минимум 2, максимум 6 подов в зависимости от нагрузки).
