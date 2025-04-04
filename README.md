import discord
from discord.ext import commands
from discord.ui import Button, View
import random

# إعداد البوت
intents = discord.Intents.default()
intents.message_content = True
bot = commands.Bot(command_prefix="!", intents=intents)

# قاعدة بيانات مؤقتة لتخزين أرصدة المستخدمين (يمكنك استخدام قاعدة بيانات حقيقية)
user_balances = {}

# التسجيل عند بدء البوت
@bot.event
async def on_ready():
    print(f'Logged in as {bot.user}')

# أمر سلاش للاطلاع على الرصيد
@bot.slash_command(name="balance", description="عرض رصيدك")
async def balance(ctx):
    user_balance = user_balances.get(ctx.user.id, 0)
    await ctx.respond(f'رصيدك الحالي هو: {user_balance} دولار')

# أمر سلاش لتحويل الأموال (للإدارة فقط)
@bot.slash_command(name="admin_give", description="إعطاء مبلغ لعضو")
async def admin_give(ctx, amount: int, user: discord.User):
    if not ctx.author.guild_permissions.administrator:
        await ctx.respond("ليس لديك صلاحيات كافية!")
        return
    if user.id not in user_balances:
        user_balances[user.id] = 0
    user_balances[user.id] += amount
    await ctx.respond(f'تم إعطاء {amount} دولار لـ {user.mention}. رصيدهم الآن: {user_balances[user.id]}')

# أمر سلاش للعب X-O
@bot.slash_command(name="xoxo", description="اللعب مع شخص آخر في X-O")
async def xoxo(ctx, opponent: discord.User, amount: int):
    if amount < 500:
        await ctx.respond("يجب أن يكون المبلغ أكبر من 500 دولار!")
        return
    if user_balances.get(ctx.user.id, 0) < amount or user_balances.get(opponent.id, 0) < amount:
        await ctx.respond("لا تملك رصيد كافٍ للمشاركة في هذه اللعبة.")
        return

    # إنشاء زر للعبة X-O
    buttons = [
        Button(label="X", style=discord.ButtonStyle.primary, custom_id="x"),
        Button(label="O", style=discord.ButtonStyle.secondary, custom_id="o")
    ]
    view = View()
    for button in buttons:
        view.add_item(button)

    # تحديد من سيفوز
    game_result = random.choice([ctx.user.id, opponent.id])

    # معالجة النتيجة
    if game_result == ctx.user.id:
        user_balances[ctx.user.id] += amount
        user_balances[opponent.id] -= amount
        await ctx.respond(f"فزت! لقد فزت بـ {amount} دولار.")
    else:
        user_balances[opponent.id] += amount
        user_balances[ctx.user.id] -= amount
        await ctx.respond(f"خسرت! {opponent.mention} فاز بـ {amount} دولار.")

# أمر سلاش لإضافة ألعاب أخرى
@bot.slash_command(name="game", description="اللعب مع الأصدقاء في لعبة عشوائية")
async def game(ctx, opponent: discord.User):
    # لعبة عشوائية بسيطة (مثال)
    winner = random.choice([ctx.user.id, opponent.id])
    if winner == ctx.user.id:
        await ctx.respond(f"لقد فزت في اللعبة مع {opponent.mention}!")
    else:
        await ctx.respond(f"{opponent.mention} فاز في اللعبة!")

# بدء البوت
bot.run('YOUR_BOT_TOKEN')
