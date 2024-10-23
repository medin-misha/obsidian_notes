Всё очень просто для этого нужно только токен бота который ты засунеш в переменную
```python
import asyncio
from core import settings
from core.handlers import start_router
from aiogram import Bot, Dispatcher
from aiogram.fsm.storage.memory import MemoryStorage
  
async def main():
	bot = Bot(token=settings.token)
	dp = Dispatcher(storage=MemoryStorage())
	  
	dp.include_routers(start_router)
	  
	await dp.start_polling(bot, polling_timeout=10**100, close_bot_session=True)

  
if __name__ == "__main__":
	asyncio.run(main())
```
Это main файл моего телеграм бота. 
- `bot`: Создает объект бота с использованием токена из настроек.
- `dp`: Создает объект диспетчера с памятью для хранения данных в памяти (`MemoryStorage`).
- `dp.include_routers(start_router)`: Включает маршрутизатор `start_router` в диспетчер для обработки команд.
- `await dp.start_polling(...)`: Запускает бот в режиме опроса, ожидая сообщения, с указанными настройками (большой таймаут и автоматическое закрытие сессии бота).
в settings/settings файле у меня такой код
```python
from pydantic_settings import BaseSettings
from dotenv import load_dotenv
import os
  
load_dotenv()

  
class Settings(BaseSettings):
	token: str
	server: str
	parse_mode: str = "HTML"
  
	class Config:
		env_file = ".env"

  
settings = Settings()
```

вот handlers/start/handler
```python
from aiogram import Router
from aiogram.types import Message
from aiogram.filters import Command
  
from .messages import texts
from .keyboard import start_keyboard
router = Router()
  
  
@router.message(Command("start"))
async def start(msg: Message):
	await msg.answer(
	text=texts.get("start").format(msg.from_user.username), reply_markup=start_keyboard())
```
а это core/handlers/start/keyboards
```python
from aiogram.utils.keyboard import ReplyKeyboardBuilder
from .messages import texts


def start_keyboard():
    builder = ReplyKeyboardBuilder()
    builder.button(text=texts.get("profile_button"))
    return builder.as_markup()
```
вот это очень простой бот.