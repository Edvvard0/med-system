<div align="center">

# 🏥 med-system

**Медицинская информационная система (МИС): регистрация пациентов, госпитализация и направления на процедуры**

![Python](https://img.shields.io/badge/Python-3.12-3776AB?style=for-the-badge&logo=python&logoColor=white)
![FastAPI](https://img.shields.io/badge/FastAPI-009688?style=for-the-badge&logo=fastapi&logoColor=white)
![PostgreSQL](https://img.shields.io/badge/PostgreSQL-4169E1?style=for-the-badge&logo=postgresql&logoColor=white)
![SQLAlchemy](https://img.shields.io/badge/SQLAlchemy-async-D71F00?style=for-the-badge)
![Docker](https://img.shields.io/badge/Docker-2496ED?style=for-the-badge&logo=docker&logoColor=white)

</div>

## 📖 О проекте

Веб-приложение и REST API для управления потоком пациентов в клинике: от регистрации и выпуска медкарты с QR-кодом до госпитализации и назначения лечебно-диагностических мероприятий. У пациентов и врачей раздельная авторизация (JWT), интерфейс построен на Jinja2-шаблонах.

> Развитие проекта [med_services](https://github.com/Edvvard0/med_services): добавлены PostgreSQL, миграции Alembic, учёт коек и перемещений, Docker Compose.

## ✨ Возможности

- **Пациенты** — регистрация (ФИО, паспорт, страховой полис и т.д.), UUID медкарты, загрузка фото, генерация и распознавание QR-кода медкарты
- **Документы** — договор и согласие на обработку данных в `.docx` по шаблонам (`docxtpl`)
- **Госпитализация** — запись на госпитализацию и просмотр по коду (диагноз, цель, отделение)
- **Лечебно-диагностические мероприятия** — назначение процедур, кабинеты, прикрепление файлов и голосовых комментариев
- **Врачи** — регистрация, вход, личный кабинет, работа с медкартой пациента
- **Коечный фонд** — занятие и освобождение мест в палатах стационара
- **Перемещения по больнице** — страница положения пациентов и сотрудников (пока на тестовых данных, см. [Ограничения](#-планы-и-ограничения))
- **Веб-интерфейс** — вход, списки пациентов / госпитализаций / процедур, профили, график процедур

## 🧱 Стек

| Технология | Роль |
|---|---|
| Python 3.12, FastAPI, Uvicorn | API и веб-сервер |
| SQLAlchemy 2 (async) + asyncpg | работа с PostgreSQL |
| Alembic | миграции БД |
| Jinja2 + HTML/CSS/JS | серверный рендеринг интерфейса |
| python-jose, passlib | JWT-авторизация, хеширование паролей |
| qrcode, OpenCV | генерация и распознавание QR-кодов |
| docxtpl, python-docx | генерация `.docx`-документов |
| pydantic-settings | конфигурация из `.env` |
| Docker Compose | окружение: приложение + PostgreSQL 15 + Redis 7 |

## 🗂 Структура проекта

```
app/
├── main.py              # FastAPI-приложение, подключение роутеров
├── config.py, database.py, logger.py, exception.py
├── dao/                 # базовый DAO-слой
├── patients/            # пациенты: модели, схемы, роутер, JWT, QR, docx
├── doctors/             # врачи: регистрация и авторизация
├── hospitalization/     # госпитализации
├── med_procedure/       # процедуры, кабинеты, файлы, голосовые комментарии
├── location/            # койки и перемещения по больнице
├── pages/               # HTML-страницы (Jinja2)
├── templates/, static/  # шаблоны, JS, шаблоны документов, загрузки
├── migration/           # Alembic
└── seed_db.py           # наполнение БД тестовыми данными
docx/                    # openapi.yml, endpoints.md, презентация об архитектуре
```

## 🚀 Быстрый старт

Сначала создайте файл `.env` в корне проекта (переменные — в разделе ниже).

**Docker Compose** (приложение + PostgreSQL + Redis):

```bash
docker compose up --build
```

Контейнер приложения сам выполняет `alembic upgrade head` и запускает Uvicorn на порту `8000`.

**Локально** (нужен запущенный PostgreSQL):

```bash
git clone https://github.com/Edvvard0/med_system.git
cd med_system
pip install -r requirements.txt     # либо: uv sync
alembic upgrade head
uvicorn app.main:app --reload
```

Тестовые данные (необязательно):

```bash
python -m app.seed_db
```

Swagger UI: http://127.0.0.1:8000/docs

## ⚙️ Конфигурация

Переменные читаются из `.env` в корне проекта (`app/config.py`); файла-примера в репозитории нет, все переменные обязательны.

| Переменная | Назначение |
|---|---|
| `DB_URL` | строка подключения SQLAlchemy (драйвер `asyncpg`) |
| `API_HOST`, `API_PORT` | хост и порт API |
| `LOG_LEVEL` | уровень логирования |
| `SECRET_KEY` | секрет для подписи JWT |
| `ALGORITHM` | алгоритм подписи JWT |

## 🔌 API

Полная спецификация — [`docx/openapi.yml`](docx/openapi.yml), описание на русском — [`docx/endpoints.md`](docx/endpoints.md).

| Префикс | Что внутри |
|---|---|
| `/patients` | регистрация, `login`, `me`, фото, QR-код, договор и согласие (`.docx`) |
| `/doctors` | список, регистрация, `login`, `me` |
| `/hospitalizations` | список, запись пациента, госпитализации пациента |
| `/med_procedure` | процедуры, кабинеты, загрузка файлов и голосовых комментариев |
| `/location` | страница перемещений, список палат, привязка и освобождение койки |
| `/pages` | HTML-интерфейс (вход пациента и врача, списки, профили, график) |

## 🗺 Планы и ограничения

- Данные о перемещениях (`app/location/utils.py`) пока берутся из заглушки; запрос к внешнему сервису закомментирован.
- Схема БД: [dbdiagram.io](https://dbdiagram.io/d/670a4c8397a66db9a3bd34fa).

<!-- TODO: скриншот интерфейса -->

## 👤 Автор

**Эдвард Осипчук** — [GitHub @Edvvard0](https://github.com/Edvvard0) · [Telegram @Edward0076](https://t.me/Edward0076)
