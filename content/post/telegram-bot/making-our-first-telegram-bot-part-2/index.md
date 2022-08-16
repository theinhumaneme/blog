+++
categories = ["telegram", "python", "telegram-bot"]
date = 2021-10-13T13:32:57Z
description = ""
draft = false
slug = "making-our-first-telegram-bot-part-2"
tags = ["telegram", "python", "telegram-bot"]
title = "Making our First Telegram Bot [Part -2]"
series = ["telegram-bot"]
[cover]
    image = "images/telegram-bot-2-1.png"
    



+++


In this Post we will be discussing the ways of making a bot, and making a simple Bot for ourselves

# How to make a Bot

Making a Telegram bot can be as easy as using a **Bot Building bots** on Telegram or by coding/scripting a Bot for ourselves which is preferable as it gives us a lot of flexibility for making a bot tailored for us.

## Which programming Language to use?

You can make a bot in many different programming languagesA few of them are:

* PHP
* Node.js
* Rust
* Python
* Ruby
* Swift
* Kotlin
* Java
* Go

In this tutorial we will be using Python and Specifically the [python-telegram-bot](https://github.com/python-telegram-bot/python-telegram-bot) library.

## Pre-requisites

* A machine with Python 3.6 or higher installed
* A Text-Editor like VSCode, Atom or PyCharm
* A basic understanding of Python Programming Language

# Coding/Scripting the Bot

Before we get started let’s get the required libraries installed

```python
pip3 install python-telegram-bot
```

let’s import all the necessary libraries and add a basic logger

```python
import logging
from telegram import Bot
from telegram.ext import Updater,CommandHandler
logging.basicConfig(
    format='%(asctime)s - %(name)s - %(levelname)s - %(message)s', level=logging.INFO
)
logger = logging.getLogger()
```

If we want `DEBUG` logs we can do the following

```python
logger.setLevel(logging.DEBUG)
```

Now lets make a instance of `Bot` in our program with our unique `API_KEY` passed to it

```python
TOKEN="<-YOUR TOKEN GOES HERE->"
bot = Bot(token=TOKEN)
```

We need handlers that will handle the commands and messages, the bot receives from the user, let us define all these handlers in the `main` function, the `MessageHandler` in our case accepts all text messages from the user and`CommandHandler` accepts `/start` command and responds to them.

```python
def main():
    updater=Updater(token=TOKEN,use_context=True)
    dp = updater.dispatcher
    dp.add_handler(CommandHandler('start',start_command))
    dp.add_handler(MessageHandler(filters=Filters.text,callback=echo_message))
```

Let’s write the callback functions for the `MessageHandler` and `CommandHandler` namely `echo_message` and `start_command`

```python
def start_command(update,context):
    user_id = update.effective_user.id
    bot.send_message(chat_id=user_id,text="Hey there\nI'm blogtest19\n")

def echo_message(update,context):
    user_id = update.effective_user.id
    message = update.message
    bot.send_message(chat_id=user_id,text=message)
```

In the above mentioned code `update` is a instance of `Update` Class and the `effective_user.id` attribute is the id of the user who generated the update in our case a message or a `/start` Command

Now adding the following line to the `main` function will enable our script to fetch the latest updates from the Telegram Servers with respect to our bot

```python
updater.start_polling()
```

We can start the script by adding to make sure that the bot works only when this script in particular is run

```python
if __name__ = "__main__":
    main()
```

{{< youtube id="3ofef3j6jyg" autoplay="false">}}

**You can find this code on my [Github Repository](https://github.com/theinhumaneme/code-snippets/blob/main/python-telegram-bot/echobot.py)**_Stay tuned for the next part in this series

