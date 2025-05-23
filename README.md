import discord
from discord.ext import commands
import datetime
from discord import Permissions

TOKEN = 'SEU_TOKEN_AQUI'

bot = commands.Bot(command_prefix="!")

@bot.event
async def on_ready():
    print(f'Bot conectado como {bot.user}')

@bot.command()
async def hello(ctx):
    await ctx.send(f'Olá, {ctx.author.mention}! Eu sou um bot do Discord. Como posso te ajudar?')

@bot.command()
async def goodnight(ctx):
    await ctx.send(f'Boa noite, {ctx.author.mention}! Durma bem.')

@bot.command()
async def soma(ctx, a: int, b: int):
    resultado = a + b
    await ctx.send(f'O resultado de {a} + {b} é {resultado}.')

@bot.command()
async def help(ctx):
    help_text = """

    - `!hello`: Saudação simples.
    - `!goodnight`: Deseja boa noite.
    - `!soma <num1> <num2>`: Soma dois números.
    - `!info`: Mostra informações sobre o bot.
    - `!ping`: Responde com "Pong!".
    - `!say <mensagem>`: O bot repete a mensagem que você passar.
    - `!kick <usuário>`: Expulsar um usuário.
    - `!ban <usuário>`: Banir um usuário.
    - `!mute <usuário>`: Mudar o status de um usuário para mudo (mute).
    - `!anuncio <mensagem>`: Enviar um anúncio.
    """
    await ctx.send(help_text)

@bot.command()
async def info(ctx):
    uptime = datetime.datetime.now() - ctx.bot.user.created_at
    info_message = f"""
    **Informações sobre o Bot:**
    - Nome: {ctx.bot.user.name}
    - ID: {ctx.bot.user.id}
    - Tempo de Atividade: {str(uptime).split('.')[0]}
    - Servidores que o bot está: {len(ctx.bot.guilds)}
    """
    await ctx.send(info_message)

@bot.command()
async def ping(ctx):
    await ctx.send("Pong!")

@bot.command()
async def say(ctx, *, message: str):
    await ctx.send(message)

@bot.command()
@commands.has_permissions(kick_members=True)  # Só pode ser usado por quem tem permissão de expulsar
async def kick(ctx, member: discord.Member, *, reason=None):
    await member.kick(reason=reason)
    await ctx.send(f'{member} foi expulso. Razão: {reason}')

@bot.command()
@commands.has_permissions(ban_members=True)  # Só pode ser usado por quem tem permissão de banir
async def ban(ctx, member: discord.Member, *, reason=None):
    await member.ban(reason=reason)
    await ctx.send(f'{member} foi banido. Razão: {reason}')

@bot.command()
@commands.has_permissions(manage_messages=True)
async def mute(ctx, member: discord.Member, *, reason=None):
    muted_role = discord.utils.get(ctx.guild.roles, name="Muted")
    
    # Se não existir o cargo "Muted", cria um
    if not muted_role:
        muted_role = await ctx.guild.create_role(name="Muted", permissions=discord.Permissions(send_messages=False, speak=False))
        for channel in ctx.guild.text_channels:
            await channel.set_permissions(muted_role, send_messages=False)
        for channel in ctx.guild.voice_channels:
            await channel.set_permissions(muted_role, speak=False)

    await member.add_roles(muted_role, reason=reason)
    await ctx.send(f'{member} foi mutado. Razão: {reason}')

@bot.command()
@commands.has_permissions(manage_messages=True)
async def unmute(ctx, member: discord.Member):
    muted_role = discord.utils.get(ctx.guild.roles, name="Muted")
    if muted_role in member.roles:
        await member.remove_roles(muted_role)
        await ctx.send(f'{member} foi desmutado.')
    else:
        await ctx.send(f'{member} não está mutado.')

@bot.command()
@commands.has_permissions(administrator=True)  # Apenas administradores podem fazer anúncios
async def anuncio(ctx, *, message: str):
    announcement_channel = discord.utils.get(ctx.guild.text_channels, name="anuncios")
    if announcement_channel:
        await announcement_channel.send(f"**Anúncio:** {message}")
    else:
        await ctx.send("Canal de anúncios não encontrado.")

bot.run(TOKEN)
