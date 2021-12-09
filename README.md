import telegram as tlg
import requests
from telegram.ext import Updater, CommandHandler, MessageHandler,Filters
from telegram import ReplyKeyboardMarkup, ReplyKeyboardRemove

wrd = ['covid', 'ковид', 'короновирус']
chats = dict() #словарь с чатами
prchats = dict() #словарь с чатами
TOKEN = '2129619225:AAHVpP1DwmvWdvnr2C-l435HrtqpPd7LpWA'

def start(bot, update):
    bot.message.reply_text("Привет! Напишите мне что-нибудь!",quote = True)

#function to handle the /help command
def help(bot, update):
    bot.message.reply_text("/stats - выводит статистику по пользователям",quote = True)

def prints(group):
    string = ""
    for i in group.items():
        string += str(i[1][0]) + ":\t" + str(i[1][1]) + "\n"
    return string

def stats(bot, update):
    string = ""
    messageID = bot.effective_chat.id
    if (bot.effective_chat.type == 'group'or bot.effective_chat.type == 'supergroup') and messageID in chats:
        string += prints(chats[messageID][0])
        bot.message.reply_text(text=string, quote = True)


    elif bot.effective_chat.type == 'private':
        for i in chats.items():
            string += str(i[1][0]) +"\n"
            string += prints(i[1][1])

        bot.message.reply_text(text=string, quote=True)

def delet(bot, update) -> None:
    global stat_dict
    if bot.effective_message.left_chat_member['is_bot'] and bot.effective_message.left_chat_member['id'] == tlg.Bot(token=TOKEN).id:
        id_group = bot.effective_chat.id
        print('Bot removed from:',bot.effective_chat.title)
        del chats[id_group]

# Определяем функцию-обработчик сообщений
# У неё два параметра, сам бот и класс updater, принявший сообщение
def echo(bot, update):
    text_received = bot.message.text
    global chats
    global prchats
    chek=True

    if bot.effective_chat.type == 'private':
        for i in wrd:
            if i in bot.message.text.lower():
                bot.message.reply_text("Сообщение может содержать неподтвержденную информацию о коронавирусной инфекции.", quote=True)
                chek=False
                break
        if(chek):
            bot.message.reply_text("Напишите что-нибудь поинтереснее!", quote=True)

    if not bot.message.from_user.is_bot:
        if bot.effective_chat.type == 'group' or bot.effective_chat.type == 'supergroup':
            for i in wrd:
                if i in bot.message.text.lower():
                    id_group = bot.effective_chat.id
                    name_group = bot.effective_chat.title
                    bot.message.reply_text("Сообщение может содержать неподтвержденную информацию о коронавирусной инфекции.", quote = True)
                    chek = False
                    id_user = bot.message.from_user.id
                    name_user = bot.message.from_user.name

                    if chats.get(id_group) is None:
                        print(name_group, name_user)
                        chats[id_group] = list()
                        chats[id_group].append(name_group)
                        chats[id_group].append(dict([(id_user, list())]))
                        chats[id_group][1][id_user].append(name_user)
                        chats[id_group][1][id_user].append(1)
                        break

                    if chats[id_group][1].get(id_user) is None:
                        chats[id_group][1].update([(id_user, [name_user,1])])
                        break
                    else:
                        chats[id_group][1][id_user][1] += 1
                        chats[id_group][1] = dict(sorted(chats[id_group][1].items(), key=lambda x:x[1][1], reverse=True))
                        break
            if (chek):
                bot.message.reply_text("Напишите что-нибудь поинтереснее!", quote=True)

def main():
    # Создаём объект updater
    updater = Updater(TOKEN, use_context=True)

    # Получаем из него диспетчер сообщений.
    dispatcher = updater.dispatcher

    # Регистрируем обработчик команды "start" в диспетчере
    dispatcher.add_handler(CommandHandler("start", start))

    dispatcher.add_handler(CommandHandler("help", help))

    dispatcher.add_handler(CommandHandler("stats", stats))

    # Создаём обработчик текстовых сообщений типа Filters.text
    text_handler = MessageHandler(Filters.text & (~Filters.command), echo)

    # Регистрируем обработчик в диспетчере.
    dispatcher.add_handler(text_handler)

    del_handler = MessageHandler(Filters.status_update.left_chat_member, delet)
    dispatcher.add_handler(del_handler)
    # Запускаем цикл приема и обработки сообщений.
    updater.start_polling()

    # Ждём завершения приложения при нажатии клавиш Ctrl+C
    updater.idle()


if __name__ == "__main__":

    try:
        main()
    except KeyboardInterrupt:
        print("err")
