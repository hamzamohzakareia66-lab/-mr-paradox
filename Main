import os
import sys
import asyncio
import json
import telebot
from PIL import Image
import edge_tts
import google.generativeai as genai
from moviepy.editor import ImageClip, AudioFileClip, CompositeVideoClip

# 1. المفاتيح والبيانات الأساسية المدمجة
GEMINI_KEY = "AQ.Ab8RN6ICh5FjndEL2LMDgAYYbMZUc3aUFbjSEAltdnLMeeoGWQ"
HF_TOKEN = "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"
BOT_TOKEN = "8885987238:AAEYEVUBjGeOsul1VsYS7knCcT_CI6EX3sI"
CHAT_ID = "7017442024"
AFFILIATE_LINK = "https://www.digistore24.com/redir/482944/mrparadox/"

genai.configure(api_key=GEMINI_KEY)
bot = telebot.TeleBot(BOT_TOKEN)

INITIAL_TOKENS = 1000
current_tokens = INITIAL_TOKENS

# الأصوات النطقية الثابتة والمحددة بدقة لكل لغة لمنع تغير خامة الصوت
LANGUAGES = {
    "ar": {"name": "Arabic", "voice": "ar-EG-ShakirNeural"},
    "en": {"name": "English", "voice": "en-US-ChristopherNeural"},
    "de": {"name": "German", "voice": "de-DE-KillianNeural"},
    "es": {"name": "Spanish", "voice": "es-ES-AlvaroNeural"},
    "fr": {"name": "French", "voice": "fr-FR-HenriNeural"},
    "pt": {"name": "Portuguese", "voice": "pt-BR-AntonioNeural"},
    "it": {"name": "Italian", "voice": "it-IT-DiegoNeural"},
    "ru": {"name": "Russian", "voice": "ru-RU-DmitryNeural"},
    "ja": {"name": "Japanese", "voice": "ja-JP-KeitaNeural"}
}

completed_tasks = []
sent_message_ids = []

# 2. المدير الذكي (Gemini Manager): يحدد المهمة القادمة بحسب التوكنز المتبقية
def ask_gemini_manager(remaining_tokens, completed_list):
    model = genai.GenerativeModel('gemini-1.5-flash')
    prompt = f"""
    You are the Master Business Manager for Mr. Paradox Automation.
    The system currently has {remaining_tokens} tokens/capacity for today.
    Completed tasks today: {completed_list}

    Available tasks:
    1. 'CREATE_VIDEOS': Generate 18 short videos (9 languages) [Cost: 400]
    2. 'CREATE_BLOG_POST': Generate high-converting SEO Blog article for affiliate link [Cost: 150]
    3. 'CREATE_PINTEREST_PINS': Generate 5 viral Pinterest pins content [Cost: 100]
    4. 'CREATE_TWITTER_THREAD': Generate a viral X/Twitter thread [Cost: 80]
    5. 'CREATE_EBOOK_LEAD': Generate a short digital guide/eBook outline [Cost: 200]

    Rules:
    - If 'CREATE_VIDEOS' is not in completed_list and remaining_tokens >= 400, choose 'CREATE_VIDEOS'.
    - Otherwise pick the highest revenue extra task that fits remaining_tokens.
    - If remaining_tokens < 80 or all done, return 'FINISH'.

    Return strictly raw JSON format without markdown code blocks:
    {{
      "next_action": "CREATE_VIDEOS" | "CREATE_BLOG_POST" | "CREATE_PINTEREST_PINS" | "CREATE_TWITTER_THREAD" | "CREATE_EBOOK_LEAD" | "FINISH",
      "estimated_cost": 100,
      "manager_note": "سبب اختيار هذه الخطوة وكيف ستدر دخلاً"
    }}
    """
    res = model.generate_content(prompt)
    clean_text = res.text.replace("```json", "").replace("```", "").strip()
    return json.loads(clean_text)

# 3. توليد الصوت ومونتاج الفيديو
async def create_audio(text, voice_name, output_filename):
    communicate = edge_tts.Communicate(text, voice_name)
    await communicate.save(output_filename)

def build_video(audio_path, image_path, output_path):
    audio_clip = AudioFileClip(audio_path)
    image_clip = ImageClip(image_path).set_duration(audio_clip.duration)
    video = CompositeVideoClip([image_clip.set_position('center')]).set_size((1080, 1920))
    video = video.set_audio(audio_clip)
    video.write_videofile(output_path, fps=24, codec='libx264', audio_codec='aac', logger=None)
    audio_clip.close()
    video.close()

# 4. إنتاج الـ 18 فيديو للـ 9 لغات
def generate_18_videos():
    model = genai.GenerativeModel('gemini-1.5-flash')
    prompt = """
    Generate 2 video concepts for 9 languages (ar, en, de, es, fr, pt, it, ru, ja) for character 'Mr. Paradox'.
    Return strictly raw JSON format:
    {
      "video_1": {"ar": {"script": "...", "caption": "..."}, "en": {"script": "...", "caption": "..."}, "de": {"script": "...", "caption": "..."}, "es": {"script": "...", "caption": "..."}, "fr": {"script": "...", "caption": "..."}, "pt": {"script": "...", "caption": "..."}, "it": {"script": "...", "caption": "..."}, "ru": {"script": "...", "caption": "..."}, "ja": {"script": "...", "caption": "..."}},
      "video_2": {"ar": {"script": "...", "caption": "..."}, "en": {"script": "...", "caption": "..."}, "de": {"script": "...", "caption": "..."}, "es": {"script": "...", "caption": "..."}, "fr": {"script": "...", "caption": "..."}, "pt": {"script": "...", "caption": "..."}, "it": {"script": "...", "caption": "..."}, "ru": {"script": "...", "caption": "..."}, "ja": {"script": "...", "caption": "..."}}
    }
    """
    res = model.generate_content(prompt)
    data = json.loads(res.text.replace("```json", "").replace("```", "").strip())
    
    image_file = "mr_paradox.png"
    if not os.path.exists(image_file):
        img = Image.new('RGB', (1080, 1920), color=(20, 20, 30))
        img.save(image_file)

    counter = 1
    for video_key in ["video_1", "video_2"]:
        for lang_code, lang_info in LANGUAGES.items():
            script_text = data[video_key][lang_code]["script"]
            caption_text = data[video_key][lang_code]["caption"]
            
            audio_path = f"temp_{lang_code}.mp3"
            video_path = f"video_{counter}_{lang_code}.mp4"
            
            asyncio.run(create_audio(script_text, lang_info["voice"], audio_path))
            build_video(audio_path, image_file, video_path)
            
            caption = (
                f"🎬 **Mr. Paradox - [{lang_info['name']}] (فيديو {counter}/18)**\n\n"
                f"{caption_text}\n\n🔗 **رابط الأفلييت (ضعْه في Bio):** {AFFILIATE_LINK}"
            )
            with open(video_path, 'rb') as v_file:
                msg = bot.send_video(CHAT_ID, v_file, caption=caption, parse_mode="Markdown")
                sent_message_ids.append(msg.message_id)
            
            if os.path.exists(audio_path): os.remove(audio_path)
            if os.path.exists(video_path): os.remove(video_path)
            counter += 1

# 5. إنتاج المهام الإضافية لزيادة الأرباح
def generate_extra_monetization_task(task_type):
    model = genai.GenerativeModel('gemini-1.5-flash')
    prompt = f"""
    Create a complete content asset for task type: '{task_type}' promoting affiliate link: {AFFILIATE_LINK}.
    Include step-by-step Arabic instructions on how to monetize this immediately.
    Return strictly raw JSON format:
    {{
      "title": "...",
      "content": "...",
      "how_to_monetize": "..."
    }}
    """
    res = model.generate_content(prompt)
    data = json.loads(res.text.replace("```json", "").replace("```", "").strip())
    
    msg_text = (
        f"💰 **مهمة دخل إضافية ({task_type}):**\n\n"
        f"📌 **العنوان:** {data['title']}\n\n"
        f"📝 **المحتوى:**\n{data['content'][:1200]}...\n\n"
        f"🚀 **إزاي تجيب منها فلوس فوراً:**\n{data['how_to_monetize']}"
    )
    msg = bot.send_message(CHAT_ID, msg_text, parse_mode="Markdown")
    sent_message_ids.append(msg.message_id)

# 6. الحلقة التنفيذية الرئيسية
def run_autonomous_workflow():
    global current_tokens, completed_tasks
    bot.send_message(CHAT_ID, f"🔄 **بداية التشغيل:** السيرفر بيتصل بـ Gemini لاستلام خطة اليوم...\n📊 الرصيد التقديري: `{current_tokens}` توكنز.")
    
    while True:
        decision = ask_gemini_manager(current_tokens, completed_tasks)
        action = decision["next_action"]
        cost = decision["estimated_cost"]
        note = decision["manager_note"]
        
        if action == "FINISH":
            bot.send_message(
                CHAT_ID, 
                f"🏁 **انتهت جميع مهام اليوم!**\n\n"
                f"📊 **المتبقي:** `{current_tokens}` توكنز.\n"
                f"✅ **المهام التي تم إنجازها:** {', '.join(completed_tasks)}\n\n"
                f"عند الانتهاء من رفع المحتوى، أرسل كلمة **'ok'** أو **'تم'** لحذف الرسائل وتوليد دفعة جديدة."
            )
            break
            
        bot.send_message(
            CHAT_ID, 
            f"🧠 **قرار المدير (Gemini):**\n"
            f"🎯 **المهمة التالية:** `{action}`\n"
            f"💡 **السبب:** {note}\n"
            f"📉 **التكلفة التقديرية:** `{cost}` توكنز."
        )
        
        if action == "CREATE_VIDEOS":
            generate_18_videos()
        else:
            generate_extra_monetization_task(action)
            
        completed_tasks.append(action)
        current_tokens -= cost
        if current_tokens < 0: current_tokens = 0
        
        bot.send_message(CHAT_ID, f"✅ اكتملت مهمة `{action}` بنجاح!\n📊 الرصيد المتبقي: `{current_tokens}` توكنز.")

# 7. الاستجابة لأوامر التلجرام وحذف الرسائل القديمة
@bot.message_handler(func=lambda message: True)
def handle_messages(message):
    global current_tokens, completed_tasks, sent_message_ids
    txt = message.text.lower().strip()
    
    if txt in ["/start", "ابدا", "ابدأ"]:
        current_tokens = INITIAL_TOKENS
        completed_tasks = []
        run_autonomous_workflow()
        
    elif txt in ["ok", "تم", "جاهز"]:
        bot.send_message(CHAT_ID, "🗑️ جاري تنظيف الرسائل السابقة وإعادة الضبط...")
        for msg_id in sent_message_ids:
            try: bot.delete_message(CHAT_ID, msg_id)
            except: pass
        sent_message_ids.clear()
        
        current_tokens = INITIAL_TOKENS
        completed_tasks = []
        run_autonomous_workflow()

if __name__ == "__main__":
    print("🤖 السيستم يعمل ومستعد لاستقبال الأوامر من التلجرام...")
    bot.infinity_polling()
