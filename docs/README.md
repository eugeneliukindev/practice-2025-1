# Документация

- Папка для размещения документации по практике в формате Markdown.
- README.md — основной файл с документацией, описывающий процесс выполнения практики.
- При необходимости могут добавляться дополнительные файлы Markdown.

Отчет о прохождении практики
Тема: Разработка Telegram-бота на фреймворке Aiogram

1. Введение
   В ходе практики была поставлена задача создать универсальный шаблон Telegram-бота с использованием современного
   асинхронного фреймворка Aiogram 3.x. Основные цели проекта:

- Снижение времени на начальную настройку новых ботов

- Внедрение лучших практик разработки

- Создание масштабируемой архитектуры

2. Этапы разработки

- Установлен Python 3.12 версии
- Настроена виртуальная среда с помощью Poetry
- Подключены ключевые зависимости:

```toml
[tool.poetry.dependencies]
aiogram = "^3.0"
sqlalchemy = "^2.0"
redis = "^4.5"
pydantic = "^2.0"
...
```
3. Проектирование архитектуры
Разработана структура проекта:
```
📦 project-root
├── 📂 app
│   ├── 📂 core
│   │   ├── 📂 models
│   │   │   └── 📂 mixins
│   │   └── 📂 schemas
│   ├── 📂 handlers
│   ├── 📂 middlewares
│   ├── 📂 repository
│   └── 📂 utils
├── 📂 docs
│   └── 📂 images
├── 📂 migrations
│   └── 📂 versions
├── 📂 scripts
├── 📂 tests
│   ├── 📂 integration_tests
│   │   ├── 📂 mock_data
│   │   └── 📂 test_handlers
│   └── 📂 unit_tests
└── 📂 texts
```
4. Реализация ключевых функций

```python
from aiogram import Bot, Dispatcher

async def main():
    bot = Bot(token=config.TOKEN)
    dp = Dispatcher()
    await dp.start_polling(bot)
...
```

5. Реализованы ORM модели SQLAlchemy для работы с БД:
```python
from sqlalchemy import BigInteger
from sqlalchemy.orm import Mapped, mapped_column

class UserOrm(BaseOrm):
    __tablename__ = 'users'
    id: Mapped[int] = mapped_column(primary_key=True)
    tg_id: Mapped[int] = mapped_column(BigInteger, unique=True)
...
```
6. Создан паттерн репозиторий для взаимодействия с БД:
```python
class UserRepository(BaseRepository):
    model_class: type[UserOrm] = UserOrm

    @classmethod
    async def get_by_tg_id(cls, session: AsyncSession, tg_id: int) -> UserOrm | None:
        scalar_result: ScalarResult[UserOrm] = await cls._get_by_fields(session=session, tg_id=tg_id)
        user: UserOrm | None = scalar_result.one_or_none()
        return user
...
```
7. Создан обработчик для регистрации пользователя через команду /start
```python
from aiogram import Router
from aiogram.filters import CommandStart
from aiogram.types import Message
from app.repository.user import UserRepository
from sqlalchemy.ext.asyncio import AsyncSession


@router.message(CommandStart())
async def command_start_handler(message: Message, session: AsyncSession) -> None:
   if await UserRepository.get_by_tg_id(session=session, tg_id=message.from_user.id):
      await message.reply("already_registered")
      return
   )
   await UserRepository.create(
      session = session,
      tg_id = message.from_user.id,
      username = message.from_user.username,
      first_name = message.from_user.first_name,
      last_name = message.from_user.last_name,
   )
   await message.reply(f"Welcome {message.from_user.full_name}!")
...
```
8. Проект собран в Docker контейнер

## В результате практики:

- Освоены современные подходы к разработке Telegram-ботов
- Приобретен опыт работы с асинхронными ORM
- Создана универсальная кодовая база для будущих проектов