---
layout: post
title: This is how you should (or shouldn't) make discord bots
description: Some tips and tricks for making discord bots
summary: All the mistakes I made and the things I learned while making discord bots
tags: coding discord bot
minute: 10
---

I have now been creating Discord bots for almost a year, and I have learned many things along the way. This is the culmination of my journey as a Discord bot creator. Throughout the past year, I have dedicated countless hours to learning, experimenting, and implementing various functionalities to make my bots efficient and user-friendly. From understanding the Discord API to integrating helpful features and commands, it has been an exhilarating experience. Along the way, I have faced challenges, debugged countless errors, and celebrated small victories.

While this article is not primarily a beginner's guide, it can still be valuable to those starting their journey in bot development. If you're new to this field and willing to ask questions (even to AI like chatgpt) or Google things you don't understand, you may find these insights, learnings, and tips beneficial. Seasoned developers might also discover some helpful techniques and best practices. So whether you're just getting started or looking to enhance your skills, this article aims to share my learnings, insights, and tips that I have gathered during this incredible journey.
# Table of Contents

1. Introduction
2. Framework
3. Getting Started
4. Setup and Configuration
   - Virtual Environment (VENV)
   - Environment Variables
5. Storing Data
6. Handling Users and Servers
7. Hosting
   - What is Docker?
   - Dockerfile Example for a Discord Bot
   - Environment Variables
   - Docker Compose
   - Commands
8. Handling Requirements
9. Importing Your Bot
10. Pycord-related Tricks
   - Command Errors
   - Default Permissions
   - Subcommands and Commands Names and Options
11. Conclusion

## Framework

There are countless frameworks for building Discord bots in many languages. I personally choose *Python* as a language and *py-cord* as a framework. Python allows me, with its easy syntax, to focus more on what I want to do than how to do it. Pycord is one of the best Python Discord frameworks out there, forked from *discord.py*. You should definitely check [this comparison](https://libs.advaith.io) to see the most used libraries in many languages.

## Getting Started

This is not a getting started tutorial, but I think it's good to add a getting started section anyways for anyone who would like to create their first bot. My advice is to first read this full article, even without fully understanding everything. Then, install Python if you don't have it already, and check the pycord guide at [guide.pycord.dev](https://guide.pycord.dev). This will allow you not to make my mistakes when you get started creating your very first bot.

## Setup and Configuration

### Virtual Environment (VENV)

When I start making a Discord bot, I usually create a Python [*venv*](https://docs.python.org/3/library/venv.html) in the folder I want to create my bot into. I then activate the environment. This depends on your operating system; you can check how to do it for yours in the docs above. It allows you to keep all the necessary dependencies and libraries for the project separate from your system's environment. This is especially useful when working on multiple projects with different requirements.

### Environment Variables

Environment variables are a standardized way of storing short strings of data, like API keys, separately from the code. At my beginnings, I used to either store my Discord token in a text file or into the codebase directly. But this is not the safest, and I eventually once got an API key pushed on GitHub. Environment variables are a great way of avoiding these mistakes and an overall practical way of storing data and settings. To use environment variables, you have two main ways: setting them in the shell or in your computer's environment variables, or having a library load them from a file. Usually, I have them in a file on the development side and set through Docker on the server side. The file is usually called `.env`, and its basic structure looks like this:

```env
DISCORD_TOKEN=token_here
some_api_key=SOME_API_KEY_HERE
etc...
```

To load these files in Python, you will need to have the `python-dotenv` library installed. Then simply use it as follows:

```python
import os
from dotenv import load_dotenv

load_dotenv()

myvalue = os.getenv("DISCORD_TOKEN")
```

The `.env` file will need to be placed into the same folder as the file you will run.

## Storing Data

I mainly use an SQL database for storing user data. If your app doesn't have lots of users, then an SQLite database will be enough; if not, you may want to use a proper cloud-hosted or self-hosted database. However, this is not very practical. That's why I prefer using a *context manager* that I import from another file, as well as server and users classes, which we will see below. Here is an example of using a database with a context manager:

**Context manager (saved to sqlConnector.py):**

```python
from sqlite3 import connect
from random import randint

class SQLConnection:
    def __init__(self, connection):
        self.connection = connection

    def __enter__(self):
        return self.connection

    def __exit__(self, exc_type, exc_val, exc_tb):
        self.connection.commit()
        self.connection.close()

class _sql:
    @property
    def mainDb(self):
        s = connect("./example.db")
        return SQLConnection(s)

sql: _sql = _sql()
```

**Actual code:**

```python
from sqlConnector import sql

# Create a table named 'users' with columns 'id', 'name', and 'age'
create_table_command = '''
CREATE TABLE IF NOT EXISTS users (
    id INTEGER PRIMARY KEY AUTOINCREMENT,
    name TEXT NOT NULL,
    age INTEGER NOT NULL
)
'''

with sql.mainDb as db:
    db.execute(create_table_command)

# Insert some data into the 'users' table
insert_data_command1 = "INSERT INTO users (name, age) VALUES ('Alice', 30)"
insert_data_command2 = "INSERT INTO users (name, age) VALUES ('Bob', 25)"

with sql.mainDb as db:
    db.execute(insert_data_command1)
    db.execute(insert_data_command2)

# Query all the data from the 'users' table and print it out
query_command = "SELECT * FROM users"

with sql.mainDb as db:
    db.execute(query_command)
    rows = db.fetchall()
    for row in rows:
        print(row)
```
## Handling Users and Servers

If you are building a Discord bot, you will probably need to perform actions repetitively for the server, or guild as we call them, in question. To do this, a good way is to use classes. I am not going to fill this page with examples, but I am sure that chatgpt will be able to give you some really nice examples or implementations.
## Hosting
I host all of my bots in a VPS, a *Virtual Private Server*. You can get one for around $3 to $4 a month, but you can also host your bots on an old computer you keep at home. I like hosting my bots with Docker because it allows me to easily handle requirements and updates.
### What is Docker?

Docker is a platform designed to simplify the process of developing, shipping, and running applications. By packaging your application and its dependencies into a "container," it ensures that your app runs the same way everywhere, whether it's on your local development machine, a test environment, or a production server.
### Dockerfile Example for a Discord Bot

Here's a simple Dockerfile that sets up a Python 3.9 environment:

```Dockerfile
# Use the official Python 3.9 image
FROM python:3.9
WORKDIR /app
COPY . .
RUN pip install --no-cache-dir -r requirements.txt
CMD ["python", "your_app.py"]
```
### Environment Variables

Environment variables can be very easily passed into your Docker container. To do this, simply create a `.env` file as explained earlier, then use the code below which will load them automatically.
### Docker Compose
Here's a `docker-compose.yml` file that builds the Docker image and sets up a volume to persist data across container restarts, without exposing any ports, since it's for a Discord bot:

```yaml
version: '3'
services:
  botator:
    build:
      context: .
      dockerfile: Dockerfile
    restart: always
    volumes:
      - ./database:/Botator/database #here I define where the data that I want to keep across updates is stored.
    env_file:
      - .env
```
### Commands

Build the image:

```bash
docker compose build --no-cache
```

Start the bot with Docker Compose:

```bash
docker compose up -d
```
## Handling Requirements

Requirements, which in our case will be the external libraries we will be using in our bot, can be defined in a file called `requirements.txt` in the root of our project. This is useful because it will allow everyone to install them with one command if needed, including in the Docker container. Your file might look like this:

```requirements
python-dotenv
py-cord
#etc...
```

Then, you can install all of it with this command:

```bash
pip install -r requirements.txt
```
## Importing Your Bot

I know this may seem obvious to some of you, but you can import your bot. Usually, I define my bot in a file called `config.py` like this, along with the bot token:

```python
import discord
import os

from dotenv import load_dotenv

load_dotenv()

discord_token = os.getenv("SUPER_SECRET_TOKEN")
bot = discord.Bot()
```

Then, I run it from another file:

```python
from config import bot, discord_token
#some stuff like loading cogs / other
#...
bot.run(discord_token)
```

This allows me to have proper and nicer code.
## Pycord-related Tricks

In this final category, we will talk about some useful Discord API or pycord features that you should DEFINITELY be using:
### Command Errors

Sometimes, for your or the user's fault, commands can throw an error. The problem is that you will never know, and the user will also never know which error occurred. Except if you tell it. That's why I am using the pycord `on_application_command_error` statement. You can easily add it to your own code in the main file and even personalize it with a link to your support server:

`````python
@bot.event
async def on_application_command_error(ctx, error: discord.DiscordException):
    message = f"""# Oh No :/ an error occurred
If you see this message, please copy the following logs and join our support discord server at https://discord.NOTgg/this_is_not_a_link

```
{str(error)}
```
"""
    await ctx.respond(message, ephemeral=True)
    raise error
`````

This will automatically be sent to the user every time an error occurs with one of your slash commands.
### Default Permissions

Sometimes, some of your commands may be a bit risky, and you would like to allow only people with some permissions to run them. Well, this can easily be done through the default permissions, and if an admin wants to allow more people to use it, it can do it directly in his server's settings through the integrations tab. Here is an example of how to do it:

```python
import discord
from discord import default_permissions

from config import bot

@bot.slash_command()
@default_permissions(admin=True)
def highlyRiskyCommand(ctx):
    #some risky code
```

This command will then show up ONLY to admins.
### Subcommands and Commands Names and Options

Setting subcommands, which allows you to add spaces in commands names, is a great idea since it allows the user to have a way cleaner experience. Setting commands names and descriptions, as well as the same for options, also allows you to have a really nice and clean experience. I will not be explaining how to do it directly here since it is specified in the Discord guide I referenced above with great examples, but I thought it was important to mention.
## Conclusion

Creating a Discord bot is an exciting and rewarding journey filled with challenges, learning, and creativity. The insights, tools, and best practices shared in this article can guide you on your path to becoming a proficient bot developer. Remember, the key is to experiment, learn from mistakes, and keep growing. Happy coding!

If you found this article helpful and want to dive deeper into the world of tech, why not join a like-minded community? **Things** is a private Discord server where enthusiasts geek out over physics, electronics, programming, AI, and more. It's a place for in-depth tech chats, away from the noise, and with a serious, mature, and respectful community. Feel free to [join us](https://link.paillat.dev/thingspycordarticle) and exchange ideas, share advancements, or just enjoy some tech banter. See you there!