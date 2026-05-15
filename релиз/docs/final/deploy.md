# Инструкция по развёртыванию
## Веб-платформа для бронирования и управления турами

---

## 1. Требования к серверу

| Компонент | Минимальная версия |
|-----------|-------------------|
| Docker | 20.10+ |
| Docker Compose | 2.0+ |
| Git | 2.0+ |
| Оперативная память | 2 GB |
| Дисковое пространство | 10 GB |

---

## 2. Пошаговая настройка

### 2.1. Клонирование репозитория

```bash
git clone https://github.com/2Dbaca/Tour_Platform.git
cd Tour_Platform
```

### 2.2. Запуск контейнеров

```bash
docker-compose up -d
```

### 2.3. Проверка работы

```bash
docker-compose ps
```

### 2.4. Доступ к сервисам

| Сервис | URL | Примечание |
|--------|-----|------------|
| Сайт | http://localhost | Основное приложение |
| phpMyAdmin | http://localhost:8080 | Управление БД (сервер: `db`, логин: `root`, пароль: `rootpassword`) |

### 2.5. Тестовые учётные записи

| Роль | Логин | Пароль |
|------|-------|--------|
| Администратор | admin | admin123 |
| Клиент | user1 | password123 |

## 3. Резервное копирование

### 3.1. Резервное копирование дампа БД

```bash
docker exec -t $(docker ps -qf "name=db") mysqldump -uroot -prootpassword tour_platform > backup_$(date +%Y%m%d).sql
```

### 3.2. Восстановление из дампа

```bash
docker exec -i $(docker ps -qf "name=db") mysql -uroot -prootpassword tour_platform < backup_20250101.sql
```

*© 2026 Веб-платформа для бронирования и управления турами*
