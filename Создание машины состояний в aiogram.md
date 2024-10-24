Для того что бы создать FSM машину в [[создание телеграмм бота в aiogram|aiogram]] нужно при создании бота в Dispatcher передать MemoryStorage()
```python
from aiogram.fsm.storage.memory import MemoryStorage
dp = Dispatcher(storage=MemoryStorage())
```
воот. Теперь создадим теперь в файле с handler-ами нужно создать состояния:
```python
from aiogram.fsm.state import State, StatesGroup


class GetUserData(StatesGroup):
    age = State()
    weight = State()
    gender = State()
```
далее нужно содать сами обработчики:
```python
@router.message(Command("profile"))
async def create_user_profile(msg: Message, state: FSMContext):
    await msg.answer(text=text.get("create_profile"))
    await state.set_state(GetUserData.age)

@router.message(F.text, GetUserData.age)
async def set_age(msg: Message, state: FSMContext):
    await state.set_data({"data": msg.text})
    await msg.answer(text=text.get("age_success"))
    await state.set_state(GetUserData.weight)
```
всё. Мы вызываем обработчик по команде `/profile` после выполнения команды нужно будет в чат вписать какие то данные, и после ввода их получит функция set_age и запишет в state. К слову такой вид хранения данных самый трушный по тому что state доступна из любого обработчика через state.get_data().