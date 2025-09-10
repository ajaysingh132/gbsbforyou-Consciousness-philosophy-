# -*- coding: utf-8 -*-
"""
Sanskrit Vedic Study Program — Mobile-first, Free+Paid API Switching
------------------------------------------------------------------
Single-file Flask app (mobile-friendly) with:
- .env loader
- Free / Paid / Auto API selection for translation/explanation
- Offline fallback (local JSON scriptures)
- Simple PWA Manifest + Service Worker endpoints
- TTS endpoint (pyttsx3 fallback)
- SQLite for users/sessions/notes

How to use:
1) Create virtualenv and install:
   python3 -m venv venv
   source venv/bin/activate
   pip install --upgrade pip
   pip install flask flask-cors sqlalchemy python-dotenv pyttsx3 googletrans==4.0.0-rc1
   # Optional (paid APIs): pip install openai requests

2) Create a .env file (see below) and place in project root.
3) Run: python Sanskrit_Vedic_Study_Program.py
4) Open on mobile browser at http://<device-ip>:5000

.env (recommended keys):
OPENAI_API_KEY=
DEEPL_API_KEY=
USE_API_MODE=auto   # values: free / paid / auto
FREE_TRANSLATE=googletrans  # reserved for extension
SECRET_KEY=some_random_string

Notes:
- Free mode uses googletrans (may be rate-limited or unreliable). Paid mode is a stub calling DeepL/OpenAI when configured.
- This is a starting platform; add authoritative scripture JSON data under data/scriptures.json.
"""

from flask import Flask, request, jsonify, render_template_string, send_from_directory
from flask_cors import CORS
import sqlite3, os, json, uuid
from contextlib import closing
from datetime import datetime
from dotenv import load_dotenv
import pyttsx3
from googletrans import Translator

# Optional imports for paid APIs
try:
    import requests
    import openai
except Exception:
    requests = None
    openai = None

# --------------------------
# Load .env
# --------------------------
load_dotenv()
OPENAI_API_KEY = os.environ.get('OPENAI_API_KEY')
DEEPL_API_KEY = os.environ.get('DEEPL_API_KEY')
USE_API_MODE = os.environ.get('USE_API_MODE', 'auto').lower()  # free / paid / auto
SECRET_KEY = os.environ.get('SECRET_KEY', 'devsecret')

# configure OpenAI if available
if openai and OPENAI_API_KEY:
    openai.api_key = OPENAI_API_KEY

# --------------------------
# Config
# --------------------------
DB_PATH = 'vedic_study.db'
DATA_DIR = 'data'
SCRIPTURES_FILE = os.path.join(DATA_DIR, 'scriptures.json')
TTS_ENABLED = True

app = Flask(__name__)
app.config['SECRET_KEY'] = SECRET_KEY
CORS(app)
translator = Translator()

if TTS_ENABLED:
    try:
        tts_engine = pyttsx3.init()
        tts_engine.setProperty('rate', 150)
    except Exception:
        tts_engine = None

# --------------------------
# DB helpers
# --------------------------

def init_db():
    os.makedirs(DATA_DIR, exist_ok=True)
    with closing(sqlite3.connect(DB_PATH)) as conn:
        c = conn.cursor()
        c.execute('''CREATE TABLE IF NOT EXISTS users (id TEXT PRIMARY KEY, name TEXT, mother_tongue TEXT, created_at TEXT)''')
        c.execute('''CREATE TABLE IF NOT EXISTS sessions (id TEXT PRIMARY KEY, user_id TEXT, scripture TEXT, prompt TEXT, result TEXT, created_at TEXT)''')
        c.execute('''CREATE TABLE IF NOT EXISTS notes (id TEXT PRIMARY KEY, session_id TEXT, note TEXT, created_at TEXT)''')
        conn.commit()


def db_execute(query, params=(), fetch=False):
    with closing(sqlite3.connect(DB_PATH)) as conn:
        c = conn.cursor()
        c.execute(query, params)
        if fetch:
            return c.fetchall()
        conn.commit()

init_db()

# --------------------------
# Load local scriptures (offline fallback)
# --------------------------
DEFAULT_SCRIPTURES = [
    {"id": "gita_2_47", "book":"gita", "chapter":2, "verse":47,
     "sanskrit":"कर्मण्येवाधिकारस्ते मा फलेषु कदाचन । मा कर्मफलहेतुर्भूर्मा ते सङ्गोऽस्त्वकर्मणि ।",
     "source":"Bhagavad Gita 2.47", "translations": {"hi":"कर्म करने में ही तेरा अधिकार है, उसके फलों में कभी नहीं; इसलिए तू कर्म-फलों का कारण नहीं बन; और न ही तुझमें अकर्मण्यता का आसक्ति हो।"}},
    {"id": "rigveda_1_1", "book":"rigveda", "hymn":1,
     "sanskrit":"अग्निम् इले पुरोहितम् यज्ञस्य देवम् ऋत्विजम् ।",
     "source":"Rigveda 1.1", "translations": {"en":"I praise Agni, the priest, god of the sacrifice, the ministrant of the ritual."}}
]

if not os.path.exists(SCRIPTURES_FILE):
    os.makedirs(DATA_DIR, exist_ok=True)
    with open(SCRIPTURES_FILE, 'w', encoding='utf-8') as f:
        json.dump(DEFAULT_SCRIPTURES, f, ensure_ascii=False, indent=2)

with open(SCRIPTURES_FILE, 'r', encoding='utf-8') as f:
    SCRIPTURES = json.load(f)

# Helper: simple search

def find_in_scriptures(query):
    q = query.lower()
    results = []
    for item in SCRIPTURES:
        if q in json.dumps(item, ensure_ascii=False).lower():
            results.append(item)
    return results

# --------------------------
# API Manager: Free / Paid / Auto
# --------------------------

def translate_free(text, dest_language):
    try:
        res = translator.translate(text, dest=dest_language)
        return res.text
    except Exception as e:
        return {'error': f'free_error: {e}', 'text': text}


def translate_paid(text, dest_language):
    # Try DeepL first (example), then OpenAI as fallback
    if DEEPL_API_KEY and requests:
        try:
            payload = {'auth_key': DEEPL_API_KEY, 'text': text, 'target_lang': dest_language.upper()}
            r = requests.post('https://api.deepl.com/v2/translate', data=payload, timeout=10)
            if r.status_code == 200:
                data = r.json()
                return data['translations'][0]['text']
        except Exception:
            pass
    if openai and OPENAI_API_KEY:
        try:
            prompt = f"Translate the following Sanskrit (or text) to {dest_language}:

{text}

Provide only the translation."
            resp = openai.ChatCompletion.create(model='gpt-4o', messages=[{'role':'user','content':prompt}], max_tokens=500)
            return resp['choices'][0]['message']['content'].strip()
        except Exception:
            pass
    return {'error': 'paid_unavailable', 'text': text}


def translate_text(text, dest_language, mode=None):
    mode = (mode or USE_API_MODE or 'auto').lower()
    if mode == 'free':
        return translate_free(text, dest_language)
    elif mode == 'paid':
        return translate_paid(text, dest_language)
    else:  # auto
        free_res = translate_free(text, dest_language)
        if isinstance(free_res, dict) and free_res.get('error'):
            paid_res = translate_paid(text, dest_language)
            return paid_res
        return free_res

# Explanation using LLM (paid) or simple template (free)

def explain_text(text, lang='hi', mode=None):
    mode = (mode or USE_API_MODE or 'auto').lower()
    if mode == 'paid' and openai and OPENAI_API_KEY:
        try:
            prompt = f"Explain this Sanskrit verse in {lang} in simple language, include literal translation and short commentary.

Verse:
{text}"
            resp = openai.ChatCompletion.create(model='gpt-4o', messages=[{'role':'user','content':prompt}], max_tokens=800)
            return resp['choices'][0]['message']['content'].strip()
        except Exception:
            pass
    # free/simple explanation: return a basic translation + note
    trans = translate_text(text, lang, mode='free')
    return f"Translation ({lang}): {trans}

[Note: For richer commentary enable Paid mode]"

# --------------------------
# TTS helper
# --------------------------

def tts_save_audio(text, filename=None):
    if not tts_engine:
        return None
    if not filename:
        filename = f"speech_{uuid.uuid4().hex[:8]}.mp3"
    path = os.path.join(DATA_DIR, filename)
    try:
        tts_engine.save_to_file(text, path)
        tts_engine.runAndWait()
        return path
    except Exception:
        return None

# --------------------------
# Frontend (mobile-first) + PWA skeleton
# --------------------------
INDEX_HTML = '''
<!doctype html>
<html lang="hi">
<head>
  <meta charset="utf-8">
  <meta name="viewport" content="width=device-width,initial-scale=1">
  <title>Vedic Study — मोबाइल</title>
  <link href="https://cdn.jsdelivr.net/npm/tailwindcss@2.2.19/dist/tailwind.min.css" rel="stylesheet">
  <meta name="theme-color" content="#0ea5a4" />
  <link rel="manifest" href="/manifest.json">
  <style>body{font-family:system-ui, -apple-system, 'Segoe UI', Roboto, 'Noto Sans', 'Helvetica Neue'}</style>
</head>
<body class="bg-gray-50 p-4">
  <div class="max-w-3xl mx-auto">
    <h1 class="text-2xl font-semibold mb-2">Vedic Study — मोबाइल</h1>
    <div class="bg-white p-3 rounded shadow mb-3">
      <label class="block text-sm">नाम</label>
      <input id="name" class="w-full border p-2 rounded" placeholder="आपका नाम (optional)" />
      <label class="block text-sm mt-2">मातृभाषा (ISO)</label>
      <input id="mother" class="w-full border p-2 rounded" placeholder="hi, en, mr, ta" />
      <div class="flex mt-2">
        <button class="flex-1 bg-teal-500 text-white p-2 rounded mr-1" onclick="createUser()">प्रोफ़ाइल बनाएं</button>
        <select id="mode" class="border p-2 rounded">
          <option value="auto">Auto (Default)</option>
          <option value="free">Free</option>
          <option value="paid">Paid</option>
        </select>
      </div>
      <div id="userResult" class="text-xs mt-2 text-gray-600"></div>
    </div>

    <div class="bg-white p-3 rounded shadow mb-3">
      <label class="block text-sm">शास्त्र</label>
      <select id="scripture" class="w-full border p-2 rounded">
        <option value="gita">गीता</option>
        <option value="upanishad">उपनिषद</option>
        <option value="veda">वेद</option>
        <option value="any">खोज (any)</option>
      </select>
      <label class="block text-sm mt-2">प्रॉम्प्ट / श्लोक</label>
      <textarea id="prompt" class="w-full border p-2 rounded" rows="4" placeholder="उदा: गीता 2:47 का अर्थ बताइए या कर्मण्येवाधिकारस्ते"></textarea>
      <label class="block text-sm mt-2">अनुवाद भाषा (ISO)</label>
      <input id="lang" class="w-full border p-2 rounded" placeholder="hi, en, mr" />
      <div class="flex mt-3">
        <button class="flex-1 bg-indigo-600 text-white p-2 rounded mr-1" onclick="sendPrompt()">अध्ययन शुरू करें</button>
        <button class="bg-gray-200 p-2 rounded" onclick="loadSample()">Sample</button>
      </div>
    </div>

    <div id="result" class="bg-white p-3 rounded shadow min-h-24"></div>

    <div class="text-xs text-gray-500 mt-3">Offline fallback available. Paid mode uses configured API keys.</div>
  </div>

<script>
let currentUser = null;
function createUser(){
  const name = document.getElementById('name').value || 'अतिथि';
  const mother = document.getElementById('mother').value || 'hi';
  const mode = document.getElementById('mode').value || 'auto';
  fetch('/api/create_user',{method:'POST',headers:{'Content-Type':'application/json'},body:JSON.stringify({name,mother,mode})})
    .then(r=>r.json()).then(j=>{currentUser=j; document.getElementById('userResult').innerText = 'User created: '+j.id})
}

function sendPrompt(){
  const scripture = document.getElementById('scripture').value;
  const prompt = document.getElementById('prompt').value;
  const lang = document.getElementById('lang').value || 'hi';
  const mode = document.getElementById('mode').value || 'auto';
  fetch('/api/study',{method:'POST',headers:{'Content-Type':'application/json'},body:JSON.stringify({user_id: currentUser?currentUser.id:null, scripture, prompt, lang, mode})})
    .then(r=>r.json()).then(j=>{
      renderResult(j);
    });
}

function renderResult(j){
  let html = '<h2 class="text-lg font-medium">परिणाम</h2>';
  if(j.found_local && j.found_local.length){
    html += '<div class="mt-2">';
    j.found_local.forEach(f=>{
      html += `<div class="p-2 border rounded mb-2"><div class="font-semibold">${f.source}</div><div class="mt-1">${f.sanskrit}</div><div class="mt-1 text-sm text-gray-700">Translation: ${f.translation||''}</div></div>`;
    });
    html += '</div>';
  }
  if(j.explanation){
    html += `<pre class="mt-2 p-2 bg-gray-100 rounded">${j.explanation}</pre>`;
  }
  if(j.heuristic_matches && j.heuristic_matches.length){
    html += '<div class="mt-2">Heuristic Matches:<ul>';
    j.heuristic_matches.forEach(h=>{html += `<li>${h.source} — ${h.sanskrit}</li>`});
    html += '</ul></div>';
  }
  document.getElementById('result').innerHTML = html;
}

function loadSample(){
  document.getElementById('prompt').value = 'कर्मण्येवाधिकारस्ते';
  document.getElementById('scripture').value = 'gita';
}

// Register service worker (optional PWA support)
if('serviceWorker' in navigator){
  navigator.serviceWorker.register('/sw.js').catch(()=>{});
}
</script>
</body>
</html>
'''

# --------------------------
# Flask routes
# --------------------------
@app.route('/')
def index():
    return render_template_string(INDEX_HTML)

@app.route('/manifest.json')
def manifest():
    manifest_data = {
        "short_name": "VedicStudy",
        "name": "Vedic Study Mobile",
        "start_url": "/",
        "display": "standalone",
        "background_color": "#ffffff",
        "theme_color": "#0ea5a4",
        "icons": []
    }
    return jsonify(manifest_data)

@app.route('/sw.js')
def service_worker():
    sw = """
self.addEventListener('install', function(e){
  self.skipWaiting();
});
self.addEventListener('fetch', function(e){
  // basic pass-through; offline pages are served from cached resources in future
});
"""
    return app.response_class(sw, mimetype='application/javascript')

# API endpoints
@app.route('/api/create_user', methods=['POST'])
def api_create_user():
    data = request.json or {}
    name = data.get('name', 'अतिथि')
    mother = data.get('mother', 'hi')
    user_id = str(uuid.uuid4())
    db_execute('INSERT INTO users (id,name,mother_tongue,created_at) VALUES (?,?,?,?)', (user_id, name, mother, datetime.utcnow().isoformat()))
    return jsonify({'id': user_id, 'name': name, 'mother_tongue': mother})

@app.route('/api/study', methods=['POST'])
def api_study():
    data = request.json or {}
    user_id = data.get('user_id')
    scripture = data.get('scripture', 'gita')
    prompt = data.get('prompt', '')
    lang = data.get('lang', 'hi')
    mode = (data.get('mode') or USE_API_MODE or 'auto').lower()

    # 1) Try local search
    found = find_in_scriptures(prompt) if prompt else []
    response = {'scripture': scripture, 'prompt': prompt, 'found_local': [], 'heuristic_matches': []}

    for f in found:
        trans = translate_text(f.get('sanskrit',''), lang, mode=mode)
        response['found_local'].append({'id': f.get('id'), 'sanskrit': f.get('sanskrit'), 'source': f.get('source'), 'translation': trans})

    # 2) Heuristic: match by scripture type
    if not found:
        heuristic = [s for s in SCRIPTURES if scripture in (s.get('book') or '').lower()]
        response['heuristic_matches'] = heuristic

    # 3) Explanation (LLM if paid, else simple)
    explanation = explain_text(prompt if prompt else (found[0]['sanskrit'] if found else ''), lang=lang, mode=mode)
    response['explanation'] = explanation

    # 4) Save session
    if user_id:
        session_id = str(uuid.uuid4())
        db_execute('INSERT INTO sessions (id,user_id,scripture,prompt,result,created_at) VALUES (?,?,?,?,?,?)', (session_id, user_id, scripture, prompt, json.dumps(response, ensure_ascii=False), datetime.utcnow().isoformat()))
        response['session_id'] = session_id

    return jsonify(response)

@app.route('/api/tts', methods=['POST'])
def api_tts():
    data = request.json or {}
    text = data.get('text','')
    if not text:
        return jsonify({'error':'text required'}), 400
    file_path = tts_save_audio(text)
    if file_path and os.path.exists(file_path):
        return jsonify({'audio_file': file_path})
    return jsonify({'error':'TTS unavailable'}), 500

@app.route('/data/<path:fname>')
def data_files(fname):
    return send_from_directory(DATA_DIR, fname)

# --------------------------
# Run
# --------------------------
if __name__ == '__main__':
    # For direct mobile-on-device use (Termux), use host 0.0.0.0
    app.run(host='0.0.0.0', port=5000, debug=True)
