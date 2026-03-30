from pyrogram import Client, filters
from pyrogram.types import *
from pyrogram.enums import ChatType
import asyncio, json, os
from datetime import datetime

# ==========================
# CONFIG
# ==========================
API_ID = 39046546
API_HASH = "a023287036896fdd5c09f1d9e7f85006"
BOT_TOKEN = "8526039240:AAG4HtVf1gXZesdbVtLb6ahl_CSETb2znbI"
OWNER_ID = 7665911046

app = Client("mm_group_manager", api_id=API_ID, api_hash=API_HASH, bot_token=BOT_TOKEN)

# ==========================
# DATABASE
# ==========================
def load(file):
    if os.path.exists(file):
        return json.load(open(file))
    return {}

def save(file, data):
    json.dump(data, open(file, "w"))

DELETE_FILE = "delete.json"
FILTER_FILE = "filters.json"
delete_texts = load(DELETE_FILE)
filters_db = load(FILTER_FILE)
users = []
mentioning = {}
broadcast = {}

# ==========================
# START
# ==========================
@app.on_message(filters.command("start") & filters.private)
async def start_private(client, message):
    if message.from_user.id not in users:
        users.append(message.from_user.id)
    me = await client.get_me()
    buttons = InlineKeyboardMarkup([
        [InlineKeyboardButton("👥 Support Group", url="https://t.me/myanmarmanagergroup")],
        [InlineKeyboardButton("📢 Official Channel", url="https://t.me/groupmanagerchannal")],
        [InlineKeyboardButton("📨 Owner ဆက်သွယ်ရန်", url="https://t.me/Maykhin2")]
    ])
    await message.reply(
        "🌸 မင်္ဂလာပါရှင်!\n"
        "✅ Myanmar Group Manager Bot အသုံးပြုပုံနဲ့အတူ အသုံးပြုနိုင်ပါတယ်\n"
        "🛠 Command များကို Help Menu မှာ မြန်မာလိုရှင်းပြထားပါတယ်",
        reply_markup=buttons
    )

@app.on_message(filters.command("start") & filters.group)
async def start_group(client, message):
    me = await client.get_me()
    buttons = InlineKeyboardMarkup([
        [InlineKeyboardButton("➕ Add Me", url=f"https://t.me/{me.username}?startgroup=true")],
        [InlineKeyboardButton("👥 Support Group", url="https://t.me/myanmarmanagergroup")]
    ])
    await message.reply(
        "🤖 Bot အသက်ရှင်နေပါတယ်!\nGroup စီမံခန့်ခွဲရန် အသင့်ပါ ✿",
        reply_markup=buttons
    )

# ==========================
# RESOLVE USER
# ==========================
async def resolve_user(client, message, arg=None):
    if message.reply_to_message:
        return message.reply_to_message.from_user.id
    if arg:
        if arg.startswith("@"):
            try:
                user = await client.get_users(arg)
                return user.id
            except:
                return None
        try:
            return int(arg)
        except:
            return None
    return None

# ==========================
# BAN / UNBAN / MUTE / UNMUTE
# ==========================
@app.on_message(filters.command("ban") & filters.group)
async def ban(client, message):
    arg = message.text.split(None, 1)[1] if len(message.text.split())>1 else None
    uid = await resolve_user(client, message, arg)
    if not uid:
        return await message.reply("❌ User မတွေ့ပါ")
    await client.ban_chat_member(message.chat.id, uid)
    await message.reply(f"🔨 User {uid} ကို ban လုပ်ပြီးပါပြီ")

@app.on_message(filters.command("unban") & filters.group)
async def unban(client, message):
    arg = message.text.split(None, 1)[1] if len(message.text.split())>1 else None
    uid = await resolve_user(client, message, arg)
    if not uid:
        return await message.reply("❌ User မတွေ့ပါ")
    await client.unban_chat_member(message.chat.id, uid)
    await message.reply(f"✅ User {uid} ကို unban လုပ်ပြီးပါပြီ")

@app.on_message(filters.command("mute") & filters.group)
async def mute(client, message):
    arg = message.text.split(None, 1)[1] if len(message.text.split())>1 else None
    uid = await resolve_user(client, message, arg)
    if not uid:
        return await message.reply("❌ User မတွေ့ပါ")
    await client.restrict_chat_member(message.chat.id, uid, permissions=ChatPermissions())
    await message.reply(f"🔇 User {uid} ကို mute လုပ်ပြီးပါပြီ")

@app.on_message(filters.command("unmute") & filters.group)
async def unmute(client, message):
    arg = message.text.split(None, 1)[1] if len(message.text.split())>1 else None
    uid = await resolve_user(client, message, arg)
    if not uid:
        return await message.reply("❌ User မတွေ့ပါ")
    await client.restrict_chat_member(message.chat.id, uid, ChatPermissions(can_send_messages=True))
    await message.reply(f"🔊 User {uid} ကို unmute ပြန်လုပ်ပြီးပါပြီ")

# ==========================
# WELCOME / GOODBYE
# ==========================
@app.on_message(filters.new_chat_members)
async def welcome(client, message):
    group_name = message.chat.title
    for member in message.new_chat_members:
        username = f"@{member.username}" if member.username else member.first_name
        await message.reply(f"🎉 {username} ({member.first_name}) ကို {group_name} မှ ကြိုဆိုပါတယ် ✿")

@app.on_message(filters.left_chat_member)
async def goodbye(client, message):
    group_name = message.chat.title
    member = message.left_chat_member
    username = f"@{member.username}" if member.username else member.first_name
    await message.reply(f"😢 {username} ({member.first_name}) သည် {group_name} မှ ထွက်သွားပါပြီ")

# ==========================
# DELETE TEXT
# ==========================
@app.on_message(filters.command("deletetext") & filters.group)
async def add_delete(c, m):
    try:
        txt = m.text.split(" ",1)[1].lower()
        chat_id = str(m.chat.id)
        delete_texts.setdefault(chat_id, [])
        if txt in delete_texts[chat_id]:
            return await m.reply("⚠️ ရှိပြီးသားပါ")
        delete_texts[chat_id].append(txt)
        save(DELETE_FILE, delete_texts)
        await m.reply(f"✅ '{txt}' ကို block လုပ်ပြီးပါပြီ")
    except:
        await m.reply("Usage: /deletetext <စာ>")

@app.on_message(filters.command("undeletetext") & filters.group)
async def remove_delete(c, m):
    try:
        txt = m.text.split(" ",1)[1].lower()
        chat_id = str(m.chat.id)
        if txt in delete_texts.get(chat_id, []):
            delete_texts[chat_id].remove(txt)
            save(DELETE_FILE, delete_texts)
            await m.reply("✅ ပြန်ခွင့်ပြုလိုက်ပါပြီ")
        else:
            await m.reply("❌ စာမတွေ့ပါ")
    except:
        await m.reply("Usage: /undeletetext <စာ>")

@app.on_message(filters.command("deletetextlist") & filters.group)
async def list_delete(c, m):
    chat_id = str(m.chat.id)
    if not delete_texts.get(chat_id):
        return await m.reply("📭 စာမရှိပါ")
    text = "\n".join(delete_texts[chat_id])
    await m.reply(f"🚫 Block စာများ:\n{text}")

@app.on_message(filters.text & filters.group, group=1)
async def auto_delete(c, m):
    chat_id = str(m.chat.id)
    text = m.text.lower()
    for t in delete_texts.get(chat_id, []):
        if t in text:
            await m.delete()
            await m.reply(f"⚠️ {m.from_user.mention} '{t}' မခွင့်ပြုပါ")
            return

# ==========================
# FILTER
# ==========================
@app.on_message(filters.command("filter") & filters.group)
async def add_filter(c, m):
    chat_id = str(m.chat.id)
    try:
        trigger, reply = m.text.split(None,2)[1:]
    except:
        return await m.reply("Usage: /filter trigger reply")
    filters_db.setdefault(chat_id, {})
    filters_db[chat_id][trigger.lower()] = reply
    save(FILTER_FILE, filters_db)
    await m.reply("✅ filter ထည့်ပြီးပါပြီ")

@app.on_message(filters.command("filters") & filters.group)
async def list_filter(c, m):
    chat_id = str(m.chat.id)
    if not filters_db.get(chat_id):
        return await m.reply("📭 filter မရှိပါ")
    text = "\n".join([f"{k} ➜ {v}" for k,v in filters_db[chat_id].items()])
    await m.reply(text)

@app.on_message(filters.command("stopfilter") & filters.group)
async def stop_filter(c, m):
    chat_id = str(m.chat.id)
    try:
        trig = m.text.split(None,1)[1].lower()
        filters_db[chat_id].pop(trig)
        save(FILTER_FILE, filters_db)
        await m.reply("❌ filter ဖျက်ပြီးပါပြီ")
    except:
        await m.reply("Usage: /stopfilter trigger")

@app.on_message(filters.command("stopall") & filters.group)
async def stop_all(c, m):
    chat_id = str(m.chat.id)
    filters_db[chat_id] = {}
    save(FILTER_FILE, filters_db)
    await m.reply("🗑 all filters ဖျက်ပြီးပါပြီ")

@app.on_message(filters.text & filters.group, group=2)
async def auto_filter(c, m):
    chat_id = str(m.chat.id)
    text = m.text.lower()
    for k,v in filters_db.get(chat_id, {}).items():
        if k in text:
            await m.reply(v)
            return

# ==========================
# BROADCAST
# ==========================
@app.on_message(filters.private & filters.command("newpost"))
async def newpost(c, m):
    if m.from_user.id != OWNER_ID: return
    broadcast[m.from_user.id] = []
    await m.reply("📢 Post ပို့လိုက်ပါ")

@app.on_message(filters.private)
async def collect(c, m):
    if m.from_user.id in broadcast:
        broadcast[m.from_user.id].append(m)
        btn = InlineKeyboardMarkup([
            [InlineKeyboardButton("✅ Send", callback_data="send")],
            [InlineKeyboardButton("❌ Cancel", callback_data="cancel")]
        ])
        await m.reply("Preview ✔️ Send မလား?", reply_markup=btn)

@app.on_callback_query()
async def cb(c, cb):
    uid = cb.from_user.id
    if cb.data == "send":
        for u in users:
            for msg in broadcast.get(uid, []):
                try:
                    await msg.copy(u)
                except:
                    pass
        await cb.message.edit("✅ ပို့ပြီးပါပြီ")
        broadcast.pop(uid, None)
    elif cb.data == "cancel":
        broadcast.pop(uid, None)
        await cb.message.edit("❌ ဖျက်လိုက်ပါပြီ")

# ==========================
# HELP MENU
# ==========================
@app.on_callback_query()
async def settings_cb(client, cb):
    data = cb.data
    if data == "help":
        help_buttons = InlineKeyboardMarkup([
            [InlineKeyboardButton("/filter", callback_data="help_filter"),
             InlineKeyboardButton("/filters", callback_data="help_filters")],
            [InlineKeyboardButton("/stopfilter", callback_data="help_stopfilter"),
             InlineKeyboardButton("/stopall", callback_data="help_stopall")],
            [InlineKeyboardButton("/deletetext", callback_data="help_deletetext"),
             InlineKeyboardButton("/undeletetext", callback_data="help_undeletetext")],
            [InlineKeyboardButton("/ban", callback_data="help_ban"),
             InlineKeyboardButton("/unban", callback_data="help_unban")],
            [InlineKeyboardButton("/mute", callback_data="help_mute"),
             InlineKeyboardButton("/unmute", callback_data="help_unmute")],
            [InlineKeyboardButton("/adminlist", callback_data="help_adminlist"),
             InlineKeyboardButton("/all", callback_data="help_all")],
            [InlineKeyboardButton("📨 Owner ဆက်သွယ်ရန်", url="https://t.me/Maykhin2")],
            [InlineKeyboardButton("✅ Done", callback_data="help_done")]
        ])
        await cb.message.edit("❓ သင်ဘာအတွက်အကူအညီလိုပါသလဲ 👇", reply_markup=help_buttons)

    # HELP DETAILS
    elif data == "help_filter":
        await cb.message.edit("📌 /filter\nအသုံးပြုပုံ: /filter <trigger> <reply>\nဥပမာ: /filter hello မင်္ဂလာပါ → User hello လိုရေးရင် bot က 'မင်္ဂလာပါ' ပြန်ပေးမယ်")
    elif data == "help_filters":
        await cb.message.edit("📌 /filters → Active filters တွေကို list ပြပါမယ်")
    elif data == "help_stopfilter":
        await cb.message.edit("📌 /stopfilter → Filter ကိုဖျက်ပါမယ်\nUsage: /stopfilter <trigger>")
    elif data == "help_stopall":
        await cb.message.edit("📌 /stopall → Group အတွက် အကုန်လုံး filter ဖျက်မယ်")
    elif data == "help_deletetext":
        await cb.message.edit("📌 /deletetext → စာ block လုပ်မယ်\nUsage: /deletetext <စာ>")
    elif data == "help_undeletetext":

await cb.message.edit("📌 /undeletetext → Block လုပ်ထားသော စာကိုပြန်ခွင့်ပြုမယ်\nUsage: /undeletetext <စာ>")
    elif data == "help_ban":
        await cb.message.edit("📌 /ban → Reply / username / user_id နဲ့ User ကို ban လုပ်နိုင်ပါသည်\nUsage: /ban <username>")
    elif data == "help_unban":
        await cb.message.edit("📌 /unban → Reply / username / user_id နဲ့ User ကို unban လုပ်နိုင်ပါသည်")
    elif data == "help_mute":
        await cb.message.edit("📌 /mute → Reply / username / user_id နဲ့ User ကို mute လုပ်နိုင်ပါသည်")
    elif data == "help_unmute":
        await cb.message.edit("📌 /unmute → Reply / username / user_id နဲ့ User ကို unmute လုပ်နိုင်ပါသည်")
    elif data == "help_adminlist":
        await cb.message.edit("📌 /adminlist → Group ထဲမှာရှိသော admin list ပြပါမယ်")
    elif data == "help_all":
        await cb.message.edit("📌 /all → Group အားလုံးကို mention လုပ်မယ်\n/stop → Mention ရပ်မယ်")
    elif data == "help_done":
        await cb.message.edit("✅ Help menu ကို အပြီးပြုပြီးပါပြီ")
    else:
        await cb.answer("⚠️ Option မရနိုင်ပါ")

# ==========================
# SPAM AUTO MUTE
# ==========================
spam_tracker = {}
SPAM_LIMIT = 5
SPAM_INTERVAL = 5
MUTE_DURATION = 60

@app.on_message(filters.text & filters.group, group=3)
async def spam_check(client, message):
    chat_id = message.chat.id
    user_id = message.from_user.id
    now = datetime.now().timestamp()

    spam_tracker.setdefault(chat_id, {})
    spam_tracker[chat_id].setdefault(user_id, [])
    spam_tracker[chat_id][user_id].append(now)

    # keep only recent SPAM_INTERVAL seconds
    spam_tracker[chat_id][user_id] = [t for t in spam_tracker[chat_id][user_id] if now - t <= SPAM_INTERVAL]

    if len(spam_tracker[chat_id][user_id]) >= SPAM_LIMIT:
        try:
            await client.restrict_chat_member(
                chat_id, user_id,
                permissions=ChatPermissions(can_send_messages=False),
                until_date=int(now + MUTE_DURATION)
            )
            await message.reply(f"⚠️ {message.from_user.mention} Spam လုပ်နေတယ်၊ {MUTE_DURATION} စက္ကန့်အတွက် mute လုပ်လိုက်ပါတယ်")
            spam_tracker[chat_id][user_id] = []
        except:
            await message.reply(f"❌ {message.from_user.mention} ကို mute မလုပ်နိုင်ပါ")

# ==========================
# RUN
# ==========================
app.run()
