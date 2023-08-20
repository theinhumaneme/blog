+++
categories = ["python", "telegram", "telegram-bot"]
date = 2022-06-14T09:48:38Z
description = ""
draft = false
slug = "lets-make-a-music-bot-part-3"
summary = "In Part -3 of the Telegram Python Bot series, we will be building an interactive bot that will send us songs from our playlists."
tags = ["python", "telegram", "telegram-bot"]
title = "Let's make a Music Bot [Part-3]"
series = ["telegram-bot"]
[cover]
    image = "images/Blog-Posters.png"
+++


Welcome to Part - 3 in the Telegram Python Bot series; in this post, we will be building an interactive bot that will send us songs from our playlists.

## This Tutorial/Post is for educational purposes only.

# Why make a music bot?

We listen to music every day, and no one hates music; it's something that everyone enjoys while going through different emotional phases of their day and life; before music streaming became a success, music used to be sold on vinyl disks and broadcasted over the radio on Radio Stations, nowadays everyone has their personally curated playlists on various platforms. Let's discuss how we will implement some of the online streaming features.

## Options

1. Random songs from the playlist chosen `random`
2. Playlist in shuffle `shuffle`
3. Random Shuffle of all the songs available `random_shuffle`
4. Send all songs available `all`

The first three options will be based on user input, and the last is just sending all the data available.

## Storage

We have two options in which we can send songs to the user

1. Files from local storage
2. Files from Telegram servers

Using local storage to store and serve our songs is an excellent idea if songs are already stored on the machine, but it has drawbacks.

1. Takes up a lot of disk space
2. While sending many songs, bandwidth and data transfer issues may arise.

The above two problems can be solved using the files from Telegram Servers, Telegram is a chat service that uses the cloud to provide multi-device support, all of our data is stored in a data centre, and each and every file on Telegram, when uploaded, is assigned a unique `64-bit` client identifier called `file_id`, these files are stored on the servers and can be sent by referencing these `file_id`. We can upload any number of files and reference them; using this, we can overcome the two issues above; however, these `file_id` are unique to chats and bots alike. For example, we upload a file `bot1` and get `file_id` - `hexa123quad` We can't use this `file_id` to send the file from `bot2`. The bots are uniquely identified using `API TOKENS`, when a token is reset, all these files are lost, as in we lose access to the files though they might continue to exist on the servers until they are deleted as they are no longer in use, saving Telegram their server space. We'll be writing the bot script keeping this drawback in mind; this can be eliminated by using persistent data stores/spaces, which can be discussed in a later post but are currently out of scope in this post.

# Making the bot

## Requirements

* A Telegram BOT API Token
* Few songs/audio files on the local machine

## Initial Setup

### Setting up a virtual environment

Setting up a virtual environment helps us isolate our dependencies and keep track of them on a per-project basis. We can set up and activate our virtual environment on Linux-based machines using these two commands.

```bash
# python3 -m venv <- name of environment ->
python3 -m venv venv
source ./venv/bin/activate
```

### Installing dependencies

```bash
pip3 install python-telegram-bot

### List installed lib/packages
pip3 list
### Save list of installed lib/packages to file
pip3 freeze >> requirements.txt
### Install dependencies from requirements.txt
pip3 install -r requirements.txt
```

Please refer to [PART-2](https://blog.kalyanmudumby.com/making-our-first-telegram-bot-part-2/) of this series to familiarise yourself with making a simple bot that echoes our message.

## Working with Files

All music/audio files on Telegram can be identified as objects of `Audio` class in the `telegram` package, we need the `file_id` attribute of this file.

### Handling Audio files.

Using `Messagehandler` let's filter all `audio` files, and `file_id_command` to handle it.

```python
dp.add_handler(MessageHandler(Filters.audio, file_id_command))
```

Now, all audio files will be handled by this function; let's echo this message via the bot.

```python
bot.send_message(chat_id=update.effective_user.id, text=f"{update.message.audio.file_id}")
```

I'll be uploading some songs, and I will categorize them into 3 different playlists to demonstrate the multiple playlists' functionality, upload numerous songs and take the `file_id` and add them to the list `playlist`, this is a nested list, where each item in `playlist` is another list. **Make sure you change the `file_id`.**

```python
playlist = [
	["CQACAgUAAxkBAANjYqUAAWP9cuxE7EgxTvTf-DG1IPRcAAJ0BgACVhYpVYcf3_AvxxpFJAQ",],
    ["CQACAgUAAxkBAANvYqUAAXISqEMcoFaCru-FpQ7TV7s6AAJ6BgACVhYpVbItmvoqnv5_JAQ",],
    ["CQACAgUAAxkBAANxYqUAAXRA6qTTIMbAPQ_LfeQ2jeb3AAJ7BgACVhYpVSuUG82IMlJvJAQ"],
]
```

### Song Command

Using the song command, let's start to serve songs; we will provide 4 options, `random`, `shuffle`, `random_shuffle`, `all` in a keyboard fashion; Telegram provides keyboards in two ways.

1. Telegram keyboard that replaces general keyboard.
2. Inline keyboard - a keyboard that is shown in the chat itself.

We'll be using the `Inline Keyboard`, it provides a clean user experience and has callback functions. We'll be having two callbacks showing new options based on user input, to keep track of user data and use it in other functions we will be using a nested global dictionary.

1. Options callback
2. Playlist Callback

#### Inline Keyboards

Let's create the keyboard with `InlineKeyboardButton` and `InlineKeyboardMarkup`, `InlineKeyboardButton` is used to identify a button and `InlineKeyboardMarkup` is used to give information to the client for rendering the keyboard; the layout of the keyboard is similar to describing a matrix using `lists`.

```python
#Defining Keyboards
main_options = [
    [InlineKeyboardButton(text="Random", callback_data="random")],
    [InlineKeyboardButton(text="Shuffle", callback_data="shuffle")],
    [InlineKeyboardButton(text="Random Shuffle", callback_data="random_shuffle")],
    [InlineKeyboardButton(text="All", callback_data="all")],
]

playlist_option = [
    [InlineKeyboardButton(text="Playlist 1", callback_data='0')],
    [InlineKeyboardButton(text="Playlist 2", callback_data='1')],
    [InlineKeyboardButton(text="Playlist 3", callback_data='2')],
]

#Creating Markups
main_options_markup = InlineKeyboardMarkup(main_options)
playlist_option_markup = InlineKeyboardMarkup(playlist_option)
```

#### Handling Callbacks

To handle callbacks from the `InlineKeyboards` we need a dedicated handler called the `CallbackQueryHandler`, since we have multiple handlers, we mention the callbacks that we can expect using the `pattern` argument, which accepts bitwise operations.

```python
dp.add_handler(CallbackQueryHandler(options_choice, pattern="^random|shuffle|random_shuffle|all$"))
dp.add_handler(CallbackQueryHandler(playlist_choice, pattern="^0|1|2$"))
```

Flow`/song` -> `options` -> `playlists` 

Options and Playlist Callback HandlerWhen user chooses `random` or `shuffle` we provide them with `playlist` choice, for `random_shuffle` we ask for the number of songs as input and proceed to send songs. All the user choices are stored in the global dictionary.

Taking user choiceUsing a message handler and `Filters` we'll accept only messages while ignoring commands, typecast them into `int` and store them in the global dict; let's also make sure that we only process user input when required using flags such as `accept` within the dict.

```python
dp.add_handler(MessageHandler((Filters.text & (~Filters.command)), user_req))
def user_req(update, context):
    if data[update.effective_user.id]["accept"] == 0:
        data[update.effective_user.id]["num_songs"] = int(update.message.text)
        print(update.message.text)
        data[update.effective_user.id]["accept"] = 1
        send_songs(update, context)
```

Sending SongsWe will be using `send_songs` method and will be accessing all the data stored until now, to send songs as per the user choice, and use the `random` module to randomize the order of the list using the `random.shuffle` method.To merge all the playlists, we can use the `extend` method creating a new list `all` with all the songs.

```python
all = list()
# Using list comprehension
all.extend(song for play in playlist for song in play)
# regular way
# all.extend(playlist[0])
# all.extend(playlist[1])
# all.extend(playlist[2]
```

```python
def send_songs(update, context):
    if (
        data[update.effective_user.id]["option"] == "random"
        or data[update.effective_user.id]["option"] == "shuffle"
        or data[update.effective_user.id]["option"] == "random_shuffle"
    ):
        
        if data[update.effective_user.id]["option"] == "random":
            # songs = random.shuffle(playlist[data[update.effective_user.id]["playlist"]])
            songs = playlist[data[update.effective_user.id]["playlist"]]
            random.shuffle(songs)
            random_songs = songs[0:data[update.effective_user.id]["num_songs"]]
            for _ in random_songs:
                bot.send_audio(chat_id=update.effective_user.id, audio=_)
        elif data[update.effective_user.id]["option"] == "shuffle":
            songs = playlist[data[update.effective_user.id]["playlist"]]
            random.shuffle(songs)
            for _ in songs:
                bot.send_audio(chat_id=update.effective_user.id, audio=_)
        elif data[update.effective_user.id]["option"] == "random_shuffle":
            all_songs = all.copy()
            random.shuffle(all_songs)
            songs = all_songs[0 : data[update.effective_user.id]["num_songs"]]
            for _ in songs:
                bot.send_audio(chat_id=update.effective_user.id, audio=_)
    elif data[update.effective_user.id]["option"] == "all":
        for song in all:
            bot.send_audio(chat_id=update.effective_user.id, audio=song)
```

{{< youtube id="hheEYZlsMvc" autoplay="false">}}

# Conclusion

We've successfully built a bot that can send us songs stored on Telegram Servers while providing a neat user experience throughout the option selection process; using the high-level classes in `python-telegram-bot` , we can build bots faster, providing more functionality. I believe that by using the above example, you can make a great music bot based on existing features or create a new one from scratch. You can build much more interesting, functional and unique bots using this platform provided by Telegram; the only limits are our imaginations.

__****You can find the code on my [Github Repository](https://github.com/theinhumaneme/code-snippets/blob/main/python-telegram-bot/music_bot.py)****__.

Thank you for reading until the end, and see you next time; until then, happy learning~ Kalyan Mudumby



