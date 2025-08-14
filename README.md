# 🧩DOBBY COPILOT DISCORD BOT - Run Locally
Meet my dobby copilot discord bot to help you :

* Analyse your code
* Detect the errors
* Fix the errors from your code
* With a proper explanation
* Uses Dobby model 70B for whole process
* Uses your fireworks AI API key

* You can add the Bot to your discord server and start fixing codes together
* Use your own `.env` file so you don't leak your Bot token and API key - 100% safe

![WhatsApp Image 2025-08-14 at 14 53 50_14f9ef32](https://github.com/user-attachments/assets/c5fdc7f4-d58c-4309-8249-9637c735af32)

![WhatsApp Image 2025-08-14 at 14 52 51_c131ec83](https://github.com/user-attachments/assets/fa0b16b9-ec8c-476b-82b1-127c6da0b04f)

---

## Follow the guide to run the bot locally using terminal:

#### Install Python & Dependencies
- Make sure you have Python 3.10+ installed
- Then in your terminal run :
```
pip install discord.py requests
```

### Install python-dotenv
```
pip install python-dotenv
```

### Install needed packages
```
pip install discord.py python-dotenv aiohttp
```

### The Bot Code
create and file in your path named : `dobby_copilot_bot.py` and paste the below code inside it
```
import os
import aiohttp
import discord
from discord import app_commands
from dotenv import load_dotenv

load_dotenv()

DISCORD_BOT_TOKEN = os.getenv("DISCORD_BOT_TOKEN")
FIREWORKS_API_KEY = os.getenv("FIREWORKS_API_KEY")

FIREWORKS_URL = "https://api.fireworks.ai/inference/v1/chat/completions"
MODEL_NAME = "accounts/sentientfoundation/models/dobby-unhinged-llama-3-3-70b-new"

intents = discord.Intents.default()
intents.message_content = True

class DobbyBot(discord.Client):
    def __init__(self):
        super().__init__(intents=intents)
        self.tree = app_commands.CommandTree(self)
        self.user_code = None

    async def on_ready(self):
        await self.tree.sync()
        print(f"✅ Logged in as {self.user}")

async def call_dobby(prompt):
    headers = {
        "Authorization": f"Bearer {FIREWORKS_API_KEY}",
        "Content-Type": "application/json",
    }
    data = {
        "model": MODEL_NAME,
        "messages": [{"role": "user", "content": prompt}],
        "max_tokens": 800
    }
    async with aiohttp.ClientSession() as session:
        async with session.post(FIREWORKS_URL, headers=headers, json=data) as resp:
            if resp.status != 200:
                return f"❌ Error: {resp.status} - {await resp.text()}"
            result = await resp.json()
            return result["choices"][0]["message"]["content"]

bot = DobbyBot()

@bot.tree.command(name="startbot", description="Start the bot and check if it's working")
async def startbot(interaction: discord.Interaction):
    await interaction.response.send_message(" Dobby Copilot Bot is online and ready!")

@bot.tree.command(name="pastecode", description="Paste your broken code")
async def pastecode(interaction: discord.Interaction, code: str):
    bot.user_code = code
    await interaction.response.send_message(" Code saved. Now run `/runcheckup` to analyze it.")

@bot.tree.command(name="runcheckup", description="Analyze the pasted code for bugs")
async def runcheckup(interaction: discord.Interaction):
    if not bot.user_code:
        await interaction.response.send_message("⚠️ No code pasted yet. Use `/pastecode` first.")
        return
    await interaction.response.defer()  # prevents timeout
    analysis = await call_dobby(f"Find and explain bugs in the following code:\n{bot.user_code}")
    await interaction.followup.send(f" **Analysis:**\n{analysis}")

@bot.tree.command(name="fixthecode", description="Fix the pasted code and explain the changes")
async def fixthecode(interaction: discord.Interaction):
    if not bot.user_code:
        await interaction.response.send_message("⚠️ No code pasted yet. Use `/pastecode` first.")
        return
    await interaction.response.defer()
    fixed = await call_dobby(f"Fix the following code and explain what was changed:\n{bot.user_code}")
    await interaction.followup.send(f"✅ **Fixed Code & Explanation:**\n{fixed}")

@bot.tree.command(name="removeapi", description="Remove the API key from memory")
async def removeapi(interaction: discord.Interaction):
    global FIREWORKS_API_KEY
    FIREWORKS_API_KEY = None
    await interaction.response.send_message("🔒 API key removed from memory.")

bot.run(DISCORD_BOT_TOKEN)
```

### Create a file called `.env`in the same folder as your `dobby_copilot_bot.py` and paste below code inside it
```
DISCORD_BOT_TOKEN=your.discord.bot.token.here
FIREWORKS_API_KEY=your.fireworks.api.key.here
```
* note: replace `your.discord.bot.token.here` with your actual Bot Token you can get it from: [Discord Dev Applications](https://discord.com/developers/applications)
* and replace `your.fireworks.api.key.here` with your actual API key you can get from: [Fireworks AI](https://app.fireworks.ai/settings/users/api-keys)

### You Are Ready To Test The Bot run:
```
python dobby_copilot_bot.py
```

---

### Now add Bot to your server and use it :
* Visit: [Discord Dev Applications](https://discord.com/developers/applications) and add bot
  
* **Features Of Bot**
* /startbot → Confirms bot is running

* /pastecode your_code_here → Saves broken code in memory

* /runcheckup → Sends code to Dobby for bug analysis

* /fixthecode → Returns fixed code + explanation

* /removeapi → Clears API key from memory

---

**Made with ❤️ by [Morsyxbt](https://x.com/morsyxbt) For [Sentient](https://x.com/SentientAGI)**
