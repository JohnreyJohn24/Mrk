import discord
from discord.ext import commands
import random
import asyncio
import datetime
import wikipedia
import requests
import json
from bs4 import BeautifulSoup
import time
import aiohttp
import os
from dotenv import load_dotenv

# Load environment variables (recommended)
load_dotenv()
TOKEN = os.getenv('DISCORD_TOKEN')  # Get token from environment variable

intents = discord.Intents.default()
intents.message_content = True
bot = commands.Bot(command_prefix="!", intents=intents)

# --- Utility Commands ---

@bot.command()
async def ping(ctx):
    """Responds with 'Pong!'"""
    await ctx.send(f"Pong! Latency: {round(bot.latency * 1000)}ms")

@bot.command()
async def invite(ctx):
    """Gets the bot's invite link."""
    await ctx.send(f"Invite me to your server: https://discord.com/api/oauth2/authorize?client_id={bot.user.id}&permissions=8&scope=bot")

@bot.command()
async def uptime(ctx):
    """Shows how long the bot has been online."""
    uptime = str(datetime.datetime.now() - bot.uptime)
    await ctx.send(f"I've been online for {uptime}")

@bot.command()
async def help(ctx, command=None):
    """Shows help information for commands."""
    if command is None:
        help_text = "**Available Commands:**\n"
        for cmd in bot.commands:
            help_text += f" - `{cmd.name}`: {cmd.help}\n"
        await ctx.send(help_text)
    else:
        cmd = bot.get_command(command)
        if cmd is not None:
            await ctx.send(f"**Help for `{cmd.name}`:**\n{cmd.help}")
        else:
            await ctx.send(f"Command `{command}` not found.")

# --- Moderation Commands ---

@bot.command()
@commands.has_permissions(kick_members=True)
async def kick(ctx, member: discord.Member, *, reason=None):
    """Kicks a member from the server."""
    await member.kick(reason=reason)
    await ctx.send(f"{member.mention} was kicked.")

@bot.command()
@commands.has_permissions(ban_members=True)
async def ban(ctx, member: discord.Member, *, reason=None):
    """Bans a member from the server."""
    await member.ban(reason=reason)
    await ctx.send(f"{member.mention} was banned.")

@bot.command()
@commands.has_permissions(unban_members=True)
async def unban(ctx, user: discord.User):
    """Unbans a user from the server."""
    await ctx.guild.unban(user)
    await ctx.send(f"{user.mention} was unbanned.")

@bot.command()
@commands.has_permissions(manage_messages=True)
async def clear(ctx, amount=5):
    """Clears a specified number of messages."""
    await ctx.channel.purge(limit=amount + 1)

@bot.command()
@commands.has_permissions(manage_messages=True)
async def slowmode(ctx, seconds: int):
    """Sets the slowmode for the current channel."""
    await ctx.channel.edit(slowmode_delay=seconds)
    await ctx.send(f"Slowmode set to {seconds} seconds.")

@bot.command()
@commands.has_permissions(manage_guild=True)
async def setprefix(ctx, new_prefix):
    """Sets a new command prefix for the bot."""
    # Store prefix in database or config file
    bot.command_prefix = new_prefix
    await ctx.send(f"Command prefix set to '{new_prefix}'.")

@bot.command()
@commands.has_permissions(manage_guild=True)
async def lockdown(ctx):
    """Locks down the current channel."""
    overwrite = ctx.channel.overwrites_for(ctx.guild.default_role)
    overwrite.send_messages = False
    await ctx.channel.set_permissions(ctx.guild.default_role, overwrite=overwrite)
    await ctx.send("Channel locked down.")

@bot.command()
@commands.has_permissions(manage_guild=True)
async def unlock(ctx):
    """Unlocks the current channel."""
    overwrite = ctx.channel.overwrites_for(ctx.guild.default_role)
    overwrite.send_messages = None
    await ctx.channel.set_permissions(ctx.guild.default_role, overwrite=overwrite)
    await ctx.send("Channel unlocked.")

@bot.command()
@commands.has_permissions(manage_guild=True)
async def mute(ctx, member: discord.Member, *, reason=None):
    """Mutes a member in the server."""
    mute_role = discord.utils.get(ctx.guild.roles, name="Muted") 
    if not mute_role:
        mute_role = await ctx.guild.create_role(name="Muted")
        for channel in ctx.guild.channels:
            await channel.set_permissions(mute_role, speak=False, send_messages=False, read_message_history=True)

    await member.add_roles(mute_role, reason=reason)
    await ctx.send(f"{member.mention} was muted.")

@bot.command()
@commands.has_permissions(manage_guild=True)
async def unmute(ctx, member: discord.Member):
    """Unmutes a member in the server."""
    mute_role = discord.utils.get(ctx.guild.roles, name="Muted")
    await member.remove_roles(mute_role)
    await ctx.send(f"{member.mention} was unmuted.")

# --- Fun Commands ---

@bot.command()
async def choose(ctx, *choices: str):
    """Chooses a random option from a list."""
    await ctx.send(f"I choose: {random.choice(choices)}")

@bot.command()
async def 8ball(ctx, *, question):
    """Answers a yes/no question with a random 8ball response."""
    responses = [
        "It is certain.", "It is decidedly so.", "Without a doubt.", 
        "Yes - definitely.", "You may rely on it.", "As I see it, yes.", 
        "Most likely.", "Outlook good.", "Yes.", "Signs point to yes.", 
        "Reply hazy, try again.", "Ask again later.", "Better not tell you now.", 
        "Cannot predict now.", "Concentrate and ask again.", 
        "Don't count on it.", "My reply is no.", "My sources say no.", 
        "Outlook not so good.", "Very doubtful."
    ]
    await ctx.send(f"Question: {question}\nAnswer: {random.choice(responses)}")

@bot.command()
async def joke(ctx):
    """Tells a random joke."""
    try:
        response = requests.get("https://some-random-api.ml/joke")
        response.raise_for_status()
        data = response.json()
        await ctx.send(f"{data['setup']}\n{data['punch_line']}")
    except:
        await ctx.send("I couldn't find a joke right now.")

@bot.command()
async def meme(ctx):
    """Sends a random meme."""
    try:
        response = requests.get("https://meme-api.herokuapp.com/gimme")
        response.raise_for_status()
        data = response.json()
        await ctx.send(data['url'])
    except:
        await ctx.send("I couldn't find a meme right now.")

@bot.command()
async def coinflip(ctx):
    """Flips a coin."""
    coin_sides = ["Heads", "Tails"]
    await ctx.send(f"You flipped a {random.choice(coin_sides)}.")

@bot.command()
async def roll(ctx, sides: int = 6):
    """Rolls a dice with the specified number of sides."""
    await ctx.send(f"You rolled a {random.randint(1, sides)}")

@bot.command()
async def fact(ctx):
    """Sends a random fact."""
    try:
        response = requests.get("https://uselessfacts.jsph.pl/
