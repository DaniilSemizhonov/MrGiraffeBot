## Welcome to MsGiraffe

The bot can record expenses and incomes and can also tell you the weather
You can write to him https://t.me/MsGiraffeBot

### Code

This is the main bot script
personal_actions.py
```markdown
from aiogram import Bot, Dispatcher, executor, types
from aiogram.dispatcher.filters import Text
from dispatcher import dp, bot
import pyowm
from pyowm.utils.config import get_default_config
import logging
import config
import re
from main import BotDB
from aiogram.types import ReplyKeyboardRemove, \
    ReplyKeyboardMarkup, KeyboardButton, \
    InlineKeyboardMarkup, InlineKeyboardButton

owm = pyowm.OWM("OWM Token")
mgr = owm.weather_manager()
#конфиги для русского языка
config_dict = get_default_config()
config_dict['language'] = 'ru'

button1 = KeyboardButton('Погода')
button2 = KeyboardButton('Бухгалтерия')

buttonuss = KeyboardButton('Москва')
buttonvla = KeyboardButton('Владивосток')
buttonrestart = KeyboardButton('Назад')

buttondo = KeyboardButton('💵Доход')
buttonras = KeyboardButton('💸Расход')
buttonhis = KeyboardButton('💰История')
buttonrestart = KeyboardButton('Назад')

rasbutton50 = KeyboardButton('💸Потратил  50')
rasbutton100 = KeyboardButton('💸Потратил 100')
rasbutton150 = KeyboardButton('💸Потратил 150')
rasbutton200 = KeyboardButton('💸Потратил 200')
rasbutton500 = KeyboardButton('💸Потратил 500')
rasbutton1000 = KeyboardButton('💸Потратил 1000')
buttonrestart = KeyboardButton('Назад')

dosbutton100 = KeyboardButton('💵Заработал 100')
dosbutton200 = KeyboardButton('💵Заработал 200')
dosbutton500 = KeyboardButton('💵Заработал 500')
dosbutton1000 = KeyboardButton('💵Заработал 1000')
dosbutton2000 = KeyboardButton('💵Заработал 2000')
dosbutton5000 = KeyboardButton('💵Заработал 5000')
buttonrestart = KeyboardButton('Назад')

markupstart = ReplyKeyboardMarkup(resize_keyboard=True).row(
    button1, button2
)
markupweather = ReplyKeyboardMarkup(resize_keyboard=True).row(
    buttonuss, buttonvla, buttonrestart
)
markupbyx = ReplyKeyboardMarkup(resize_keyboard=True).row(
    buttondo, buttonras, buttonhis, buttonrestart
)
markupras = ReplyKeyboardMarkup().add(
    rasbutton50, rasbutton100, rasbutton150, rasbutton200, rasbutton500, rasbutton1000, buttonrestart
)
markupdos = ReplyKeyboardMarkup().add(
    dosbutton100, dosbutton200, dosbutton500, dosbutton1000, dosbutton2000, dosbutton5000, buttonrestart
)

@dp.message_handler(commands='start')
async def start_cmd_handler(message: types.Message):
    if (not BotDB.user_exists(message.from_user.id)):
        BotDB.add_user(message.from_user.id)
    await message.answer("Доброго времяни суток:) \nЯ - MsGiraffe, бот который може посчитать ваши доходы и расходы, подсказать погоду и уведомит о новой статье на любимом сайте. Бот Создан @DaniilSemizhonov"
                     ,reply_markup=markupstart)

@dp.message_handler(Text(equals="Назад"))
async def start_cmd_handler(message: types.Message):
    await message.answer("Вернул тебя назад", reply_markup=markupstart)

@dp.message_handler(Text(equals="Москва"))
async def start_cmd_handler(message: types.Message):
    Moscow = "Москва"
    observation = mgr.weather_at_place(Moscow)
    w = observation.weather
    temp = w.temperature('celsius')['temp']

    answer = "В " + Moscow + "е" + " сейчас " + w.detailed_status
    answer += " температура: " + str(temp) + "°"

    await message.answer(answer, reply_markup=markupstart)

@dp.message_handler(Text(equals="Владивосток"))
async def start_cmd_handler(message: types.Message):
    Vladivostok = "Владивосток"
    observation = mgr.weather_at_place(Vladivostok)
    w = observation.weather
    temp = w.temperature('celsius')['temp']

    answer = "Во " + Vladivostok + "е" + " сейчас " + w.detailed_status
    answer += " температура: " + str(temp) + "°"
    await message.answer(answer, reply_markup=markupstart)

@dp.message_handler(Text(equals="Погода"))
async def with_puree(message: types.Message):
    await message.reply("Где хочешь погоду узнать", reply_markup=markupweather)

@dp.message_handler(Text(equals="Бухгалтерия"))
async def with_puree(message: types.Message):
    await message.reply("Личный бухгалтер на связи", reply_markup=markupbyx)

@dp.message_handler(Text(equals="💸Расход"))
async def with_puree(message: types.Message):
    await message.reply("Сколько составил чек?", reply_markup=markupras)

@dp.message_handler(Text(equals="💵Доход"))
async def with_puree(message: types.Message):
    await message.reply("Сколько получил?", reply_markup=markupdos)


@dp.message_handler(commands = ("Потратил", "Заработал"), commands_prefix = "💸💵")
async def start(message: types.Message):
    cmd_variants = (("💸Потратил"), ("💵Заработал"))
    operation = '-' if message.text.startswith(cmd_variants[0]) else '+'

    value = message.text
    for i in cmd_variants:
        for j in i:
            value = value.replace(j, '').strip()

    if(len(value)):
        x = re.findall(r"\d+(?:.\d+)?", value)
        if(len(x)):
            value = float(x[0].replace(',', '.'))

            BotDB.add_record(message.from_user.id, operation, value)

            if(operation == '-'):
                await message.reply("✅ Запись о <u><b>расходе</b></u> успешно внесена!", reply_markup=markupstart)
            else:
                await message.reply("✅ Запись о <u><b>доходе</b></u> успешно внесена!", reply_markup=markupstart)
        else:
            await message.reply("Не удалось определить сумму!")
    else:
        await message.reply("Не введена сумма!")


@dp.message_handler(commands = ("История"), commands_prefix = "💰")
async def start(message: types.Message):
    cmd_variants = ("История")
    within_als = {
        "day": ('today', 'day', 'сегодня', 'день'),
        "month": ('month', 'месяц'),
        "year": ('year', 'год'),
    }

    cmd = message.text
    for r in cmd_variants:
        cmd = cmd.replace(r, '').strip()

    within = 'day'
    if(len(cmd)):
        for k in within_als:
            for als in within_als[k]:
                if(als == cmd):
                    within = k

    records = BotDB.get_records(message.from_user.id, within)

    if(len(records)):
        answer = f"🕘 История операций за {within_als[within][-1]}\n\n"

        for r in records:
            answer += "<b>" + ("➖ Расход" if not r[2] else "➕ Доход") + "</b>"
            answer += f" - {r[3]}"
            answer += f" <i>({r[4]})</i>\n"

        await message.reply(answer)
    else:
        await message.reply("Записей не обнаружено!")
```


### Using the code

If you want to use the code, you need to replace the telegram bot token and the pyowm token

### Support or Contact

My telegram @DaniilSemizhonov
Email Daniil_Semizhonov@mail.ru
