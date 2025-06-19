# MagicCrossBot
import logging
from telegram import Update, InlineKeyboardButton, InlineKeyboardMarkup
from telegram.ext import (
    ApplicationBuilder,
    CommandHandler,
    CallbackQueryHandler,
    MessageHandler,
    filters,
    ContextTypes,
    ConversationHandler
)

# Настройка логирования
logging.basicConfig(
    format='%(asctime)s - %(name)s - %(levelname)s - %(message)s',
    level=logging.INFO
)
logger = logging.getLogger(__name__)

# Ваш токен бота (ЗАМЕНИТЕ НА СВОЙ!)
TOKEN = "7220977859:AAH04n1gfmH0h_ZZa2ywJDysjA5nGuaD60Y"

# Состояния для заказа
ВВОД_ОПИСАНИЯ, ПОДТВЕРЖДЕНИЕ_ЗАКАЗА = range(2)

# База данных схем
СХЕМЫ = {
    "001": {
        "название": "Милая кошечка", 
        "цена": 299, 
        "размер": "15x15 см",
        "описание": "Простая схема для начинающих с милой кошечкой",
        "цвета": "8 цветов мулине"
    },
    "002": {
        "название": "Пейзаж с рекой", 
        "цена": 399, 
        "размер": "20x15 см",
        "описание": "Живописный пейзаж с рекой и лесом",
        "цвета": "12 цветов мулине"
    },
    "003": {
        "название": "Розы в вазе", 
        "цена": 349, 
        "размер": "18x18 см",
        "описание": "Элегантный букет роз в вазе",
        "цвета": "10 цветов мулине"
    }
}

# ID администратора для уведомлений
ID_АДМИНА = "ВАШ_CHAT_ID"  # Замените на ваш ID в Telegram

# Клавиатуры
def главное_меню():
    клавиатура = [
        [InlineKeyboardButton("🛍️ Каталог схем", callback_data='каталог')],
        [InlineKeyboardButton("🎨 Индивидуальный заказ", callback_data='индивидуальный_заказ')],
        [InlineKeyboardButton("ℹ️ О магазине", callback_data='о_нас')],
        [InlineKeyboardButton("📞 Контакты", callback_data='контакты')]
    ]
    return InlineKeyboardMarkup(клавиатура)

def клавиатура_каталога():
    кнопки = []
    for код, товар in СХЕМЫ.items():
        кнопки.append([InlineKeyboardButton(
            f"{товар['название']} - {товар['цена']} руб", 
            callback_data=f"товар_{код}"
        )])
    кнопки.append([InlineKeyboardButton("🔙 Назад", callback_data='назад')])
    return InlineKeyboardMarkup(кнопки)

def клавиатура_назад():
    return InlineKeyboardMarkup([[InlineKeyboardButton("🔙 Назад", callback_data='назад')]])

# Обработчики команд
async def старт(update: Update, context: ContextTypes.DEFAULT_TYPE):
    await update.message.reply_text(
        "✨ Добро пожаловать в магазин схем для вышивки крестиком!\n"
        "Здесь вы можете купить готовые схемы или заказать индивидуальный дизайн.",
        reply_markup=главное_меню()
    )

async def помощь(update: Update, context: ContextTypes.DEFAULT_TYPE):
    await update.message.reply_text("Используйте кнопки меню для навигации.")

# Обработчики callback-запросов
async def обработка_кнопок(update: Update, context: ContextTypes.DEFAULT_TYPE):
    запрос = update.callback_query
    await запрос.answer()
    данные = запрос.data

    if данные == 'каталог':
        await показать_каталог(запрос)
    elif данные == 'индивидуальный_заказ':
        await начать_индивидуальный_заказ(запрос, context)
    elif данные == 'о_нас':
        await о_нас(запрос)
    elif данные == 'контакты':
        await контакты(запрос)
    elif данные == 'назад':
        await назад_в_меню(запрос)
    elif данные.startswith('товар_'):
        код = данные.split('_')[1]
        await показать_схему(запрос, код)
    elif данные.startswith('купить_'):
        код = данные.split('_')[1]
        await оформить_покупку(запрос, код)
    elif данные.startswith('оплата_'):
        части = данные.split('_')
        метод = части[1]
        код = части[2]
        await показать_реквизиты(запрос, метод, код)
    elif данные.startswith('подтвердить_'):
        код = данные.split('_')[1]
        await подтвердить_оплату(запрос, код)
    elif данные == 'подтвердить_заказ':
        await подтвердить_заказ(update, context)

async def показать_каталог(запрос):
    await запрос.edit_message_text(
        "🏷️ Выберите схему из каталога:",
        reply_markup=клавиатура_каталога()
    )

async def о_нас(запрос):
    await запрос.edit_message_text(
        "🎨 О нашем магазине:\n\n"
        "Мы создаем красивые и качественные схемы для вышивки с 2023 года.\n"
        "Все схемы протестированы на реальных работах и гарантируют отличный результат.\n\n"
        "Почему выбирают нас:\n"
        "✅ Качественная проработка деталей\n"
        "✅ Оптимизированная цветовая палитра\n"
        "✅ Поддержка после покупки",
        reply_markup=клавиатура_назад()
    )

async def контакты(запрос):
    await запрос.edit_message_text(
        "📞 Наши контакты:\n\n"
        "Администратор: @ваш_администратор\n"
        "Техподдержка: @ваша_поддержка\n"
        "Email: embroidery@example.com\n\n"
        "Часы работы: 10:00-20:00 (МСК)",
        reply_markup=клавиатура_назад()
    )

async def назад_в_меню(запрос):
    await запрос.edit_message_text(
        "Главное меню:",
        reply_markup=главное_меню()
    )

async def показать_схему(запрос, код):
    товар = СХЕМЫ[код]
    # В реальном боте здесь должно быть отправка изображения схемы
    # await context.bot.send_photo(chat_id=запрос.message.chat_id, photo=open(f'схемы/{код}.jpg', 'rb'))
    
    await запрос.edit_message_text(
        f"🧵 <b>{товар['название']}</b>\n\n"
        f"📏 Размер: {товар['размер']}\n"
        f"🎨 Цвета: {товар['цвета']}\n"
        f"💰 Цена: <b>{товар['цена']} руб</b>\n\n"
        f"{товар['описание']}\n\n"
        "После оплаты вы получите:\n"
        "✅ PDF-файл со схемой\n"
        "✅ Цветовую легенду\n"
        "✅ Инструкцию по вышиванию",
        parse_mode="HTML",
        reply_markup=InlineKeyboardMarkup([
            [InlineKeyboardButton("💳 Купить сейчас", callback_data=f"купить_{код}")],
            [InlineKeyboardButton("🔙 В каталог", callback_data='каталог')]
        ])
    )

async def оформить_покупку(запрос, код):
    товар = СХЕМЫ[код]
    await запрос.edit_message_text(
        f"🛒 Оформление заказа: <b>{товар['название']}</b>\n\n"
        f"Сумма к оплате: <b>{товар['цена']} руб</b>\n\n"
        "Пожалуйста, выберите способ оплаты:",
        parse_mode="HTML",
        reply_markup=InlineKeyboardMarkup([
            [InlineKeyboardButton("💳 Карта (СБП)", callback_data=f"оплата_карта_{код}")],
            [InlineKeyboardButton("🥝 QIWI Кошелек", callback_data=f"оплата_qiwi_{код}")],
            [InlineKeyboardButton("🔙 Назад", callback_data=f"товар_{код}")]
        ])
    )

async def показать_реквизиты(запрос, метод, код):
    товар = СХЕМЫ[код]
    # Данные для оплаты (замените на свои)
    реквизиты = {
        'карта': "💳 Оплата картой:\nНомер: 4890 4700 **** 1234\nПолучатель: Иван Иванов",
        'qiwi': "🥝 Оплата QIWI:\nКошелек: +79001234567\nКомментарий: 123456"
    }
    
    await запрос.edit_message_text(
        f"❇️ Для оплаты схемы <b>«{товар['название']}»</b>:\n\n"
        f"{реквизиты[метод]}\n\n"
        "После оплаты пришлите скриншот чека для подтверждения.\n\n"
        "Ваш заказ будет обработан в течение 15 минут после подтверждения оплаты.",
        parse_mode="HTML",
        reply_markup=InlineKeyboardMarkup([
            [InlineKeyboardButton("✅ Я оплатил", callback_data=f"подтвердить_{код}")],
            [InlineKeyboardButton("🔙 Назад", callback_data=f"купить_{код}")]
        ])
    )

async def подтвердить_оплату(запрос, код):
    товар = СХЕМЫ[код]
    # Здесь должно быть отправка файла (в реальном боте)
    # await context.bot.send_document(chat_id=запрос.message.chat_id, document=open(f'схемы/{код}.pdf', 'rb'))
    
    await запрос.edit_message_text(
        f"🎉 Спасибо за покупку <b>«{товар['название']}»</b>!\n\n"
        "Схема и материалы отправлены вам в чат.\n\n"
        "Если возникнут вопросы по вышиванию - обращайтесь!\n\n"
        "Хотите оставить отзыв о нашей работе?",
        parse_mode="HTML",
        reply_markup=InlineKeyboardMarkup([
            [InlineKeyboardButton("⭐ Оставить отзыв", url="https://t.me/ваш_канал")],
            [InlineKeyboardButton("🏠 В главное меню", callback_data='назад')]
        ])
    )

# Индивидуальный заказ
async def начать_индивидуальный_заказ(запрос, context):
    await запрос.edit_message_text(
        "🎨 Расскажите о вашей идее для индивидуальной схемы:\n\n"
        "Опишите:\n"
        "1. Сюжет или изображение\n"
        "2. Желаемый размер вышивки\n"
        "3. Особые пожелания\n\n"
        "Пример: \"Хочу схему по фото моей собаки, размер 20×25 см, в реалистичном стиле\"",
        reply_markup=клавиатура_назад()
    )
    return ВВОД_ОПИСАНИЯ

async def обработка_описания(update: Update, context: ContextTypes.DEFAULT_TYPE):
    описание = update.message.text
    context.user_data['описание_заказа'] = описание
    
    await update.message.reply_text(
        "📝 Ваш заказ:\n\n"
        f"{описание}\n\n"
        "Всё верно?",
        reply_markup=InlineKeyboardMarkup([
            [InlineKeyboardButton("✅ Да, всё верно", callback_data='подтвердить_заказ')],
            [InlineKeyboardButton("✏️ Нет, изменить", callback_data='изменить_заказ')]
        ])
    )
    return ПОДТВЕРЖДЕНИЕ_ЗАКАЗА

async def подтвердить_заказ(update: Update, context: ContextTypes.DEFAULT_TYPE):
    запрос = update.callback_query
    await запрос.answer()
    
    # Отправка уведомления администратору
    текст_заказа = (
        "🚨 НОВЫЙ ИНДИВИДУАЛЬНЫЙ ЗАКАЗ!\n\n"
        f"От: @{запрос.from_user.username}\n"
        f"ID: {запрос.from_user.id}\n\n"
        f"Описание:\n{context.user_data['описание_заказа']}"
    )
    
    await context.bot.send_message(
        chat_id=ID_АДМИНА,
        text=текст_заказа
    )
    
    await запрос.edit_message_text(
        "✅ Ваш заказ принят!\n\n"
        "Наш дизайнер свяжется с вами в течение 24 часов для уточнения деталей.\n\n"
        "Средний срок разработки индивидуальной схемы: 3-5 дней.\n\n"
        "Спасибо за доверие!",
        reply_markup=InlineKeyboardMarkup([
            [InlineKeyboardButton("🏠 В главное меню", callback_data='назад')]
        ])
    )
    return ConversationHandler.END

async def отменить_заказ(update: Update, context: ContextTypes.DEFAULT_TYPE):
    await update.message.reply_text(
        "Заказ отменен.",
        reply_markup=главное_меню()
    )
    return ConversationHandler.END

# Обработка текстовых сообщений
async def обработка_текста(update: Update, context: ContextTypes.DEFAULT_TYPE):
    await update.message.reply_text(
        "Используйте кнопки меню для навигации:",
        reply_markup=главное_меню()
    )

def main():
    приложение = ApplicationBuilder().token(TOKEN).build()
    
    # Обработчики команд
    приложение.add_handler(CommandHandler('start', старт))
    приложение.add_handler(CommandHandler('help', помощь))
    
    # Обработчик кнопок
    приложение.add_handler(CallbackQueryHandler(обработка_кнопок))
    
    # Обработчик индивидуальных зака
