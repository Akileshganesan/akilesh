<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>Gamified AI | Environmental Education Platform</title>
    <link href="https://cdn.jsdelivr.net/npm/bootstrap@5.3.2/dist/css/bootstrap.min.css" rel="stylesheet">
    <script src="https://cdn.tailwindcss.com"></script>
    <link rel="stylesheet" href="https://cdnjs.cloudflare.com/ajax/libs/font-awesome/6.5.1/css/all.min.css">
</head>
<body class="bg-gradient-to-br from-emerald-950 via-slate-900 to-teal-950 min-h-screen text-slate-100 flex flex-col justify-between">
    <nav class="navbar border-b border-emerald-500/20 bg-slate-950/80 backdrop-blur-md sticky-top py-3">
        <div class="container mx-auto px-4 flex justify-between items-center">
            <a class="text-2xl font-black text-emerald-400 flex items-center gap-2 text-decoration-none" href="/">
                <i class="fa-solid fa-gamepad text-emerald-400 text-3xl"></i>
                <span>Gamified<span class="text-white"> AI</span></span>
            </a>
            <div class="flex items-center gap-3">
                {% if session.get('user_id') %}
                    {% if session.get('role') == 'teacher' %}
                        <a href="/teacher" class="bg-emerald-600 text-white text-sm font-bold py-2 px-4 rounded-xl text-decoration-none">Teacher Dashboard</a>
                    {% else %}
                        <a href="/student" class="bg-teal-600 text-white text-sm font-bold py-2 px-4 rounded-xl text-decoration-none">Student Arcade</a>
                    {% endif %}
                    <a href="/leaderboard" class="bg-amber-600 text-white text-sm font-bold py-2 px-4 rounded-xl text-decoration-none">Leaderboard</a>
                    <a href="/logout" class="bg-rose-600 text-white text-sm font-bold py-2 px-4 rounded-xl text-decoration-none">Logout</a>
                {% else %}
                    <a href="/login?role=teacher" class="bg-emerald-600 hover:bg-emerald-500 text-white text-sm font-bold py-2 px-4 rounded-xl text-decoration-none">Teacher Portal</a>
                    <a href="/login?role=student" class="bg-teal-600 hover:bg-teal-500 text-white text-sm font-bold py-2 px-4 rounded-xl text-decoration-none">Student Arcade</a>
                {% endif %}
            </div>
        </div>
    </nav>
    <main class="container mx-auto px-4 py-8 flex-grow">
        {% with messages = get_flashed_messages(with_categories=true) %}
          {% if messages %}
            {% for category, message in messages %}
              <div class="alert alert-{{ category }} mb-4 rounded-xl text-center font-bold" role="alert">
                {{ message }}
              </div>
            {% endfor %}
          {% endif %}
        {% endwith %}

        {% block content %}{% endblock %}
    </main>
    <footer class="border-t border-slate-800 bg-slate-950/60 py-4 text-center text-slate-400 text-xs">
        <p>Gamified AI • Environmental Education Platform for Schools & Colleges</p>
    </footer>
</body>
</html>
{% extends 'base.html' %}
{% block content %}
<div class="space-y-6">
    <div class="bg-slate-900 p-6 rounded-3xl border border-slate-800 flex justify-between items-center shadow-xl">
        <div>
            <h2 class="text-2xl font-black text-white">{{ student.name }}</h2>
            <p class="text-xs text-emerald-400 font-bold"><i class="fa-solid fa-graduation-cap"></i> {{ student.grade_semester }} • Level {{ student.level }} Eco-Ranger</p>
        </div>
        <div class="bg-amber-500/10 border border-amber-500/30 px-4 py-2 rounded-2xl text-amber-400 font-extrabold text-lg">
            <i class="fa-solid fa-bolt"></i> {{ student.xp }} XP
        </div>
    </div>

    <h3 class="text-xl font-bold text-white">Available Quest Rooms</h3>
    <div class="grid grid-cols-1 md:grid-cols-3 gap-6">
        {% for game in games %}
        <div class="bg-slate-900 p-6 rounded-3xl space-y-3 border border-slate-800 shadow-xl flex flex-col justify-between">
            <div>
                <span class="bg-emerald-500/20 text-emerald-400 text-xs px-3 py-1 rounded-xl font-bold border border-emerald-500/30 inline-block mb-2">{{ game.grade_semester }}</span>
                <h4 class="text-lg font-bold text-white">{{ game.title }}</h4>
            </div>
            <a href="/game/{{ game.id }}" class="block text-center bg-emerald-600 hover:bg-emerald-500 text-white font-bold py-2.5 rounded-xl text-decoration-none transition">Enter Quest Room</a>
        </div>
        {% endfor %}
    </div>
</div>
{% endblock %}
<script>
let lastAiReply = "";

async function sendChatMessage() {
    const chatInput = document.getElementById('chatInput');
    const msg = chatInput.value.trim();
    const grade = document.getElementById('gradeInput').value.trim();
    if (!msg) return;

    const chatMessages = document.getElementById('chatMessages');
    
    // Add user message
    chatMessages.innerHTML += `<div class="bg-emerald-900/40 p-3 rounded-xl text-emerald-200 ml-4 font-semibold border border-emerald-500/20">Teacher: ${msg}</div>`;
    chatInput.value = '';
    
    // Add temporary loading indicator
    const tempId = "loading-" + Date.now();
    chatMessages.innerHTML += `<div id="${tempId}" class="bg-slate-800 p-3 rounded-xl border border-slate-700 text-slate-400 mr-4 font-medium animate-pulse">🤖 Thinking...</div>`;
    chatMessages.scrollTop = chatMessages.scrollHeight;

    try {
        const res = await fetch('http://127.0.0.1:3000/api/node-chat', {
            method: 'POST',
            headers: {'Content-Type': 'application/json'},
            body: JSON.stringify({message: msg, grade_semester: grade})
        });

        const data = await res.json();
        lastAiReply = data.reply;

        // Replace loading indicator with AI response
        const loader = document.getElementById(tempId);
        if (loader) loader.remove();

        chatMessages.innerHTML += `<div class="bg-slate-800 p-3 rounded-xl border border-slate-700 text-slate-100 mr-4 font-medium whitespace-pre-line">🤖 Node.js AI:\n${data.reply}</div>`;
    } catch (err) {
        console.error("Chat connection error:", err);
        const loader = document.getElementById(tempId);
        if (loader) loader.remove();

        chatMessages.innerHTML += `<div class="bg-rose-900/40 p-3 rounded-xl text-rose-300 mr-4 border border-rose-500/30">⚠️ Connection Failed: Ensure 'node server.js' is running in terminal tab 1 on Port 3000.</div>`;
    }
    chatMessages.scrollTop = chatMessages.scrollHeight;
}

function copyChatToSyllabus() {
    if (lastAiReply) {
        const syllabusBox = document.getElementById('syllabusInput');
        syllabusBox.value += (syllabusBox.value ? "\n\n" : "") + lastAiReply;
        alert("Copied AI suggestions directly into your Syllabus box!");
    } else {
        alert("No AI response to copy yet. Send a message first!");
    }
}

// Allow sending message by pressing Enter key
document.getElementById("chatInput").addEventListener("keypress", function(event) {
    if (event.key === "Enter") {
        event.preventDefault();
        sendChatMessage();
    }
});
</script>
{% extends 'base.html' %}
{% block content %}
<div class="grid grid-cols-1 lg:grid-cols-3 gap-8">
    <!-- Left Column: Syllabus Upload Form -->
    <div class="lg:col-span-2 bg-slate-900 p-8 rounded-3xl border border-slate-800 shadow-2xl space-y-6">
        <div class="border-b border-slate-800 pb-4 flex justify-between items-center">
            <div>
                <h2 class="text-2xl font-bold text-white">Teacher Portal</h2>
                <p class="text-slate-400 text-sm">Welcome, <strong>{{ teacher.name }}</strong></p>
            </div>
            <span class="bg-emerald-500/20 text-emerald-400 px-3 py-1 rounded-xl text-xs font-bold border border-emerald-500/30">
                <i class="fa-solid fa-shield-halved"></i> Teacher Verified
            </span>
        </div>

        <form action="/teacher" method="POST" class="space-y-4">
            <div>
                <label class="block font-bold text-sm mb-1 text-slate-300">Target Grade / Semester</label>
                <input type="text" id="gradeInput" name="grade_semester" placeholder="e.g. B.Tech Semester 2" required class="w-full p-3 bg-slate-800 border border-slate-700 rounded-xl text-white outline-none">
            </div>

            <div>
                <label class="block font-bold text-sm mb-1 text-slate-300">Environmental Syllabus / Topics</label>
                <textarea id="syllabusInput" name="syllabus_text" rows="5" placeholder="Paste units here or discuss with the Node.js AI Assistant..." required class="w-full p-3 bg-slate-800 border border-slate-700 rounded-xl text-white outline-none"></textarea>
            </div>

            <div>
                <label class="block font-bold text-sm mb-1 text-slate-300">Canva Game Embed Link (Optional)</label>
                <input type="url" name="canva_url" placeholder="https://www.canva.com/design/.../view?embed" class="w-full p-3 bg-slate-800 border border-slate-700 rounded-xl text-white outline-none">
            </div>

            <button type="submit" class="w-full bg-emerald-600 hover:bg-emerald-500 text-white font-bold py-3 rounded-xl transition shadow-lg">
                <i class="fa-solid fa-gamepad"></i> Generate Gamified Eco Quest
            </button>
        </form>
    </div>

    <!-- Right Column: Node.js Powered AI Chatbox -->
    <div class="bg-slate-900 p-6 rounded-3xl border border-slate-800 shadow-2xl flex flex-col h-[550px] justify-between">
        <div class="border-b border-slate-800 pb-3 flex justify-between items-center">
            <h3 class="text-base font-bold text-emerald-400 flex items-center gap-2">
                <i class="fa-brands fa-node-js text-emerald-400 text-xl"></i> Node.js AI Assistant
            </h3>
            <span class="bg-emerald-500/10 text-emerald-400 text-[10px] font-extrabold px-2 py-0.5 rounded-full border border-emerald-500/20">PORT 3000</span>
        </div>

        <div id="chatMessages" class="flex-grow overflow-y-auto space-y-3 py-3 text-xs pr-2">
            <div class="bg-slate-800 p-3 rounded-xl border border-slate-700 text-slate-300">
                ⚡ Powered by Node.js & Gemini! Ask me to suggest or structure syllabus topics for your course.
            </div>
        </div>

        <div class="space-y-2 pt-2 border-t border-slate-800">
            <div class="flex gap-2">
                <input type="text" id="chatInput" placeholder="Ask Node.js AI for syllabus topics..." class="flex-grow p-2.5 bg-slate-800 border border-slate-700 rounded-xl text-white text-xs outline-none">
                <button onclick="sendChatMessage()" class="bg-emerald-600 hover:bg-emerald-500 text-white px-4 rounded-xl text-xs font-bold transition">Send</button>
            </div>
            <button onclick="copyChatToSyllabus()" class="w-full bg-teal-600/20 hover:bg-teal-600 text-teal-300 hover:text-white border border-teal-500/30 py-2 rounded-xl text-xs font-bold transition">
                📋 Copy Suggestions to Syllabus Box
            </button>
        </div>
    </div>
</div>

<script>
let lastAiReply = "";

async function sendChatMessage() {
    const chatInput = document.getElementById('chatInput');
    const msg = chatInput.value.trim();
    const grade = document.getElementById('gradeInput').value.trim();
    if (!msg) return;

    const chatMessages = document.getElementById('chatMessages');
    chatMessages.innerHTML += `<div class="bg-emerald-900/40 p-3 rounded-xl text-emerald-200 ml-4 font-semibold">Teacher: ${msg}</div>`;
    chatInput.value = '';

    try {
        const res = await fetch('/api/teacher-chat', {
            method: 'POST',
            headers: {'Content-Type': 'application/json'},
            body: JSON.stringify({message: msg, grade_semester: grade})
        });
        const data = await res.json();
        lastAiReply = data.reply;
        chatMessages.innerHTML += `<div class="bg-slate-800 p-3 rounded-xl border border-slate-700 text-slate-200 mr-4 font-medium">🤖 Node.js AI: ${data.reply}</div>`;
    } catch (err) {
        chatMessages.innerHTML += `<div class="bg-rose-900/40 p-3 rounded-xl text-rose-300 mr-4">⚠️ Connection Error: Ensure Node.js server.js is running.</div>`;
    }
    chatMessages.scrollTop = chatMessages.scrollHeight;
}

function copyChatToSyllabus() {
    if (lastAiReply) {
        const syllabusBox = document.getElementById('syllabusInput');
        syllabusBox.value += (syllabusBox.value ? "\n" : "") + lastAiReply;
        alert("Copied Node.js AI suggestions directly into your Syllabus box!");
    }
}
</script>
{% endblock %}
{% extends 'base.html' %}
{% block content %}
<div class="grid grid-cols-1 lg:grid-cols-3 gap-8">
    <div class="lg:col-span-2 bg-slate-900 p-6 rounded-3xl border border-slate-800 shadow-2xl space-y-4">
        <div class="flex justify-between items-center">
            <h2 class="text-xl font-bold text-emerald-400">{{ game_data.adventure_title }}</h2>
            <span id="userXpBadge" class="bg-amber-500/20 text-amber-400 text-xs font-bold px-3 py-1 rounded-xl border border-amber-500/30">
                XP: {{ student.xp }}
            </span>
        </div>

        {% if game.canva_embed_url %}
        <iframe class="w-full aspect-video rounded-2xl border border-slate-700" src="{{ game.canva_embed_url }}" allowfullscreen></iframe>
        {% else %}
        <div class="p-12 text-center bg-slate-950 rounded-2xl border border-slate-800">
            <i class="fa-solid fa-earth-americas text-6xl text-emerald-400 mb-3 animate-pulse"></i>
            <h3 class="text-white font-bold text-lg">Interactive Eco-World Room</h3>
        </div>
        {% endif %}
    </div>

    <div class="space-y-4">
        {% for module in game_data.modules %}
        <div class="bg-slate-900 p-5 rounded-2xl border border-slate-800 space-y-3">
            <h4 class="font-bold text-teal-300 text-sm border-b border-slate-800 pb-2">{{ module.module_name }}</h4>
            {% for quest in module.quests %}
            <div class="bg-slate-950 p-4 rounded-xl border border-slate-800 space-y-2">
                <p class="font-bold text-sm text-white">{{ quest.title }}</p>
                <p class="text-xs text-emerald-400">Concept: {{ quest.learning_concept }}</p>
                <p class="text-xs text-slate-300">Task: {{ quest.real_world_task }}</p>
                
                <div class="mt-2 space-y-1">
                    <p class="text-xs text-slate-400 font-semibold">Q: {{ quest.quiz.question }}</p>
                    {% for opt in quest.quiz.options %}
                    <button data-answer="{{ opt }}" data-correct="{{ quest.quiz.correct_answer }}" onclick="checkAnswer(this)" class="w-full text-left bg-slate-800 hover:bg-emerald-600 p-2 text-xs rounded-lg text-slate-200 transition font-medium">
                        {{ opt }}
                    </button>
                    {% endfor %}
                </div>
            </div>
            {% endfor %}
        </div>
        {% endfor %}
    </div>
</div>

<script>
async function checkAnswer(btn) {
    const selected = btn.getAttribute('data-answer');
    const correct = btn.getAttribute('data-correct');

    if (selected === correct) {
        btn.classList.remove('bg-slate-800');
        btn.classList.add('bg-emerald-600', 'text-white', 'font-bold');

        const res = await fetch('/api/add-xp', {method: 'POST'});
        const data = await res.json();
        if (data.success) {
            document.getElementById('userXpBadge').innerText = `XP: ${data.xp}`;
            alert(`🎉 Correct Answer! +100 XP awarded! Total XP: ${data.xp}`);
        }
    } else {
        btn.classList.remove('bg-slate-800');
        btn.classList.add('bg-rose-600', 'text-white');
        alert("❌ Incorrect answer! Try again.");
    }
}
</script>
{% endblock %}
{% extends 'base.html' %}
{% block content %}
<div class="max-w-4xl mx-auto py-12 text-center space-y-6">
    <div class="inline-flex items-center gap-2 bg-emerald-500/10 border border-emerald-500/30 text-emerald-300 px-4 py-1.5 rounded-full text-sm font-semibold mb-2">
        <i class="fa-solid fa-wand-magic-sparkles text-emerald-400"></i> Gamified AI Education
    </div>
    
    <h1 class="text-5xl font-black text-white">Gamified AI Environmental Education Platform</h1>
    <p class="text-slate-300 text-lg">Turn any school or college syllabus into interactive quests, Canva games, and real-world eco challenges.</p>
    
    <div class="flex justify-center gap-6 pt-6">
        <a href="/login?role=teacher" class="bg-emerald-500 hover:bg-emerald-400 text-slate-950 font-black py-4 px-8 rounded-2xl text-decoration-none shadow-xl">
            <i class="fa-solid fa-chalkboard-user"></i> Teacher Portal Login
        </a>
        <a href="/login?role=student" class="bg-teal-500 hover:bg-teal-400 text-slate-950 font-black py-4 px-8 rounded-2xl text-decoration-none shadow-xl">
            <i class="fa-solid fa-gamepad"></i> Student Arcade Login
        </a>
    </div>
</div>
{% endblock %}
{% extends 'base.html' %}
{% block content %}
<div class="max-w-2xl mx-auto bg-slate-900 p-8 rounded-3xl border border-slate-800 shadow-2xl space-y-6">
    <h2 class="text-2xl font-black text-amber-400 text-center flex items-center justify-center gap-2">
        <i class="fa-solid fa-trophy"></i> Registered Student Ranks
    </h2>
    <p class="text-center text-xs text-slate-400">Ranks calculated strictly for students with active accounts</p>

    <div class="space-y-3">
        {% for student in students %}
        <div class="p-4 bg-slate-950 rounded-2xl border border-slate-800 flex justify-between items-center">
            <div class="flex items-center gap-3">
                <span class="w-8 h-8 rounded-full bg-slate-800 text-slate-200 text-xs font-black flex items-center justify-center">
                    #{{ loop.index }}
                </span>
                <div>
                    <span class="font-bold text-white text-sm block">{{ student.name }}</span>
                    <span class="text-xs text-emerald-400 font-semibold">{{ student.grade_semester }} • Level {{ student.level }}</span>
                </div>
            </div>
            <span class="text-amber-400 font-extrabold text-sm"><i class="fa-solid fa-bolt"></i> {{ student.xp }} XP</span>
        </div>
        {% else %}
        <p class="text-center text-slate-500 text-sm py-4">No registered students yet.</p>
        {% endfor %}
    </div>
</div>
{% endblock %}
{% extends 'base.html' %}
{% block content %}
<div class="max-w-md mx-auto bg-slate-900 text-white p-8 rounded-3xl border border-slate-800 shadow-2xl space-y-6">
    <h2 class="text-2xl font-black text-center text-emerald-400 uppercase tracking-wider">
        {{ role|title }} Login
    </h2>
    <form action="/login" method="POST" class="space-y-4">
        <input type="hidden" name="role" value="{{ role }}">
        <div>
            <label class="block text-sm font-bold mb-1">Username</label>
            <input type="text" name="username" required class="w-full p-3 rounded-xl bg-slate-800 border border-slate-700 text-white outline-none">
        </div>
        <div>
            <label class="block text-sm font-bold mb-1">Password</label>
            <input type="password" name="password" required class="w-full p-3 rounded-xl bg-slate-800 border border-slate-700 text-white outline-none">
        </div>
        <button type="submit" class="w-full bg-emerald-600 hover:bg-emerald-500 text-white font-bold py-3 rounded-xl transition">
            Login as {{ role|title }}
        </button>
    </form>
    <p class="text-center text-sm text-slate-400">
        Don't have an account? <a href="/register?role={{ role }}" class="text-emerald-400 font-bold text-decoration-none">Register here</a>
    </p>
</div>
{% endblock %}
{% extends 'base.html' %}
{% block content %}
<div class="max-w-md mx-auto bg-slate-900 text-white p-8 rounded-3xl border border-slate-800 shadow-2xl space-y-6">
    <h2 class="text-2xl font-black text-center text-emerald-400 uppercase tracking-wider">
        Register as {{ role|title }}
    </h2>
    <form action="/register" method="POST" class="space-y-4">
        <input type="hidden" name="role" value="{{ role }}">
        <div>
            <label class="block text-sm font-bold mb-1">Full Name</label>
            <input type="text" name="name" placeholder="e.g. Alex Smith" required class="w-full p-3 rounded-xl bg-slate-800 border border-slate-700 text-white outline-none">
        </div>
        <div>
            <label class="block text-sm font-bold mb-1">Username</label>
            <input type="text" name="username" required class="w-full p-3 rounded-xl bg-slate-800 border border-slate-700 text-white outline-none">
        </div>
        <div>
            <label class="block text-sm font-bold mb-1">Grade or Semester</label>
            <input type="text" name="grade_semester" placeholder="e.g. Class 10 OR B.Tech Sem 2" required class="w-full p-3 rounded-xl bg-slate-800 border border-slate-700 text-white outline-none">
        </div>
        <div>
            <label class="block text-sm font-bold mb-1">Password</label>
            <input type="password" name="password" required class="w-full p-3 rounded-xl bg-slate-800 border border-slate-700 text-white outline-none">
        </div>
        <button type="submit" class="w-full bg-emerald-600 hover:bg-emerald-500 text-white font-bold py-3 rounded-xl transition">
            Create {{ role|title }} Account
        </button>
    </form>
</div>
{% endblock %}
import json
import os
import random
from google import genai
from google.genai import types
from dotenv import load_dotenv

load_dotenv()

api_key = os.getenv("GEMINI_API_KEY")

# Initialize Gemini Client
client = None
if api_key and api_key != "your_gemini_api_key_here":
    try:
        client = genai.Client(api_key=api_key)
    except Exception as e:
        print(f"Error initializing GenAI Client: {e}")

def generate_gamified_syllabus(syllabus_text, grade_or_semester):
    # Try calling Gemini API directly
    if client:
        try:
            prompt = f"""
            You are Gemini, an expert AI Curriculum Specialist and Gamification Architect for Environmental Science & Engineering.

            A teacher/professor submitted the following syllabus for Grade/Semester: '{grade_or_semester}'.

            TEACHER SYLLABUS / TOPIC INPUT:
            "{syllabus_text}"

            INSTRUCTIONS FOR GEMINI:
            1. Thoroughly analyze the core concepts in the provided syllabus text.
            2. Generate 2 to 3 creative, distinct gaming modules directly based on these specific topics.
            3. For EACH module, generate 2 quests with:
               - A specific quest title and real-world task/experiment.
               - A unique, conceptually deep multiple-choice question (quiz) testing the syllabus concepts.
               - 4 plausible options and 1 correct answer matching one of the options EXACTLY.
            4. Make the questions deep and specific (e.g. if the topic is Water Technology, ask about hardness titration or reverse osmosis; if Electrochemistry, ask about Nernst equation or cell EMF). DO NOT return generic placeholder text.

            RETURN ONLY A VALID JSON OBJECT matching this exact schema without any markdown formatting or surrounding backticks:
            {{
              "adventure_title": "Creative Quest Title based on syllabus",
              "modules": [
                {{
                  "module_name": "Module Title from Syllabus Topics",
                  "story_line": "Immersive narrative hook connecting this topic to real-world eco impact",
                  "quests": [
                    {{
                      "quest_id": "q1",
                      "title": "Quest Title",
                      "xp_reward": 150,
                      "learning_concept": "Core Concept from Syllabus",
                      "real_world_task": "Actionable lab task or real-world eco activity",
                      "quiz": {{
                        "question": "Deep multiple choice question testing this specific concept",
                        "options": ["Option A", "Option B", "Option C", "Option D"],
                        "correct_answer": "Exact string of the correct option"
                      }}
                    }}
                  ]
                }}
              ]
            }}
            """

            response = client.models.generate_content(
                model='gemini-2.5-flash',
                contents=prompt,
                config=types.GenerateContentConfig(
                    response_mime_type="application/json"
                )
            )
            
            raw_text = response.text.strip()
            # Clean up potential markdown code block wrappers
            if raw_text.startswith("```json"):
                raw_text = raw_text[7:]
            if raw_text.startswith("```"):
                raw_text = raw_text[3:]
            if raw_text.endswith("```"):
                raw_text = raw_text[:-3]

            return json.loads(raw_text.strip())

        except Exception as e:
            print(f"⚠️ Gemini API Call Error: {e}. Falling back to Smart Context Generator.")

    # Fallback Smart Engine if API Key is missing or quota/network fails
    return generate_smart_context_syllabus(syllabus_text, grade_or_semester)


def generate_smart_context_syllabus(syllabus_text, grade_or_semester):
    """
    Intelligent context parser that breaks down the actual text input into dynamic questions 
    so you get topic-specific questions even when running offline or without an API key.
    """
    # Clean and extract topic keywords from input
    topics = [t.strip() for t in syllabus_text.replace('\n', ',').split(',') if len(t.strip()) > 3]
    
    if not topics:
        topics = ["Environmental Impact Assessment", "Sustainable Resource Management", "Eco-Engineering"]

    mod_1_topic = topics[0]
    mod_2_topic = topics[1] if len(topics) > 1 else "Renewable Technology & Ecosystem Restoration"

    return {
        "adventure_title": f"Eco-Quest: {mod_1_topic[:30]} ({grade_or_semester})",
        "modules": [
            {
                "module_name": f"Module 1: Fundamentals of {mod_1_topic[:35]}",
                "story_line": f"Dive deep into the principles of {mod_1_topic} to analyze real-world environmental and industrial data.",
                "quests": [
                    {
                        "quest_id": "q1",
                        "title": f"Lab Audit: {mod_1_topic[:25]}",
                        "xp_reward": 150,
                        "learning_concept": mod_1_topic,
                        "real_world_task": f"Analyze a case study or perform a campus audit focusing on {mod_1_topic}.",
                        "quiz": {
                            "question": f"Which of the following fundamental principles directly governs {mod_1_topic}?",
                            "options": [
                                f"Thermodynamic efficiency and sustainable control of {mod_1_topic[:20]}",
                                "Uncontrolled resource degradation without recovery",
                                "Ignoring equilibrium constraints in closed systems",
                                "Linear consumption without recycling processes"
                            ],
                            "correct_answer": f"Thermodynamic efficiency and sustainable control of {mod_1_topic[:20]}"
                        }
                    }
                ]
            },
            {
                "module_name": f"Module 2: Applied {mod_2_topic[:35]}",
                "story_line": f"Apply system-level solutions to solve challenges related to {mod_2_topic}.",
                "quests": [
                    {
                        "quest_id": "q2",
                        "title": f"Engineering Challenge: {mod_2_topic[:25]}",
                        "xp_reward": 200,
                        "learning_concept": mod_2_topic,
                        "real_world_task": f"Propose an eco-friendly engineering solution to minimize impact in {mod_2_topic}.",
                        "quiz": {
                            "question": f"What is the primary operational objective when implementing {mod_2_topic} in modern engineering?",
                            "options": [
                                f"Maximizing process efficiency while minimizing carbon/waste output in {mod_2_topic[:15]}",
                                "Increasing non-renewable energy dependency",
                                "Eliminating monitoring and safety standards",
                                "Disregarding environmental impact assessments"
                            ],
                            "correct_answer": f"Maximizing process efficiency while minimizing carbon/waste output in {mod_2_topic[:15]}"
                        }
                    }
                ]
            }
        ]
    }
    from flask import Flask, render_template, request, redirect, url_for, session, jsonify, flash
from database import db, User, SyllabusGame
from ai_engine import generate_gamified_syllabus
import json, os

app = Flask(__name__)

# Configure SQLite URI
basedir = os.path.abspath(os.path.dirname(__file__))
app.config['SQLALCHEMY_DATABASE_URI'] = 'sqlite:///' + os.path.join(basedir, 'gamified_ai.db')
app.config['SQLALCHEMY_TRACK_MODIFICATIONS'] = False
app.config['SECRET_KEY'] = 'sih_gamified_ai_secret_key_2026'

# Initialize DB with App Context
db.init_app(app)

with app.app_context():
    db.create_all()

@app.route('/')
def home():
    return render_template('index.html')

@app.route('/register', methods=['GET', 'POST'])
def register():
    if request.method == 'POST':
        username = request.form.get('username')
        name = request.form.get('name')
        password = request.form.get('password')
        role = request.form.get('role')
        grade_semester = request.form.get('grade_semester', 'General')

        if User.query.filter_by(username=username).first():
            flash('Username already exists. Please choose another.', 'danger')
            return redirect(url_for('register', role=role))

        user = User(username=username, name=name, role=role, grade_semester=grade_semester)
        user.set_password(password)
        db.session.add(user)
        db.session.commit()

        session['user_id'] = user.id
        session['role'] = user.role
        session['name'] = user.name

        if user.role == 'teacher':
            return redirect(url_for('teacher_dashboard'))
        return redirect(url_for('student_dashboard'))

    role = request.args.get('role', 'student')
    return render_template('register.html', role=role)

@app.route('/login', methods=['GET', 'POST'])
def login():
    if request.method == 'POST':
        username = request.form.get('username')
        password = request.form.get('password')
        role = request.form.get('role')

        user = User.query.filter_by(username=username, role=role).first()
        if user and user.check_password(password):
            session['user_id'] = user.id
            session['role'] = user.role
            session['name'] = user.name

            if user.role == 'teacher':
                return redirect(url_for('teacher_dashboard'))
            return redirect(url_for('student_dashboard'))
        
        flash('Invalid credentials or selected role.', 'danger')
        return redirect(url_for('login', role=role))

    role = request.args.get('role', 'student')
    return render_template('login.html', role=role)

@app.route('/logout')
def logout():
    session.clear()
    return redirect(url_for('home'))

@app.route('/teacher', methods=['GET', 'POST'])
def teacher_dashboard():
    if 'user_id' not in session or session.get('role') != 'teacher':
        return redirect(url_for('login', role='teacher'))

    teacher = User.query.get(session['user_id'])

    if request.method == 'POST':
        grade_semester = request.form.get('grade_semester', 'General Grade')
        syllabus_content = request.form.get('syllabus_text', '')
        canva_url = request.form.get('canva_url', '')

        gamified_data = generate_gamified_syllabus(syllabus_content, grade_semester)

        new_game = SyllabusGame(
            title=gamified_data.get('adventure_title', f'Gamified AI Quest ({grade_semester})'),
            grade_semester=grade_semester,
            created_by=teacher.id,
            quests_json=json.dumps(gamified_data),
            canva_embed_url=canva_url
        )
        db.session.add(new_game)
        db.session.commit()
        return redirect(url_for('student_dashboard'))

    games = SyllabusGame.query.filter_by(created_by=teacher.id).all()
    return render_template('dashboard_teacher.html', games=games, teacher=teacher)

@app.route('/student')
def student_dashboard():
    if 'user_id' not in session or session.get('role') != 'student':
        return redirect(url_for('login', role='student'))

    student = User.query.get(session['user_id'])
    games = SyllabusGame.query.all()
    return render_template('dashboard_student.html', games=games, student=student)

@app.route('/game/<int:game_id>')
def game_room(game_id):
    if 'user_id' not in session:
        return redirect(url_for('login', role='student'))

    game = SyllabusGame.query.get_or_404(game_id)
    game_data = json.loads(game.quests_json)
    student = User.query.get(session['user_id'])
    return render_template('game_view.html', game=game, game_data=game_data, student=student)

@app.route('/api/add-xp', methods=['POST'])
def add_xp():
    if 'user_id' in session and session.get('role') == 'student':
        student = User.query.get(session['user_id'])
        student.xp += 100
        student.level = (student.xp // 200) + 1
        db.session.commit()
        return jsonify({'success': True, 'xp': student.xp, 'level': student.level})
    return jsonify({'success': False})

@app.route('/leaderboard')
def leaderboard():
    students = User.query.filter_by(role='student').order_by(User.xp.desc()).all()
    return render_template('leaderboard.html', students=students)

if __name__ == '__main__':
    print("-------------------------------------------------------")
    print("🌱 Gamified AI Platform Running On: http://127.0.0.1:5000")
    print("-------------------------------------------------------")
    app.run(debug=True, port=5000)
    from flask_sqlalchemy import SQLAlchemy
from werkzeug.security import generate_password_hash, check_password_hash

db = SQLAlchemy()

class User(db.Model):
    __tablename__ = 'users'
    
    id = db.Column(db.Integer, primary_key=True)
    username = db.Column(db.String(80), unique=True, nullable=False)
    name = db.Column(db.String(100), nullable=False)
    password_hash = db.Column(db.String(200), nullable=False)
    role = db.Column(db.String(20), nullable=False)  # 'student' or 'teacher'
    grade_semester = db.Column(db.String(50))
    xp = db.Column(db.Integer, default=0)
    level = db.Column(db.Integer, default=1)

    def set_password(self, password):
        self.password_hash = generate_password_hash(password)

    def check_password(self, password):
        return check_password_hash(self.password_hash, password)

class SyllabusGame(db.Model):
    __tablename__ = 'syllabus_games'
    
    id = db.Column(db.Integer, primary_key=True)
    title = db.Column(db.String(200), nullable=False)
    grade_semester = db.Column(db.String(50), nullable=False)
    created_by = db.Column(db.Integer, db.ForeignKey('users.id'))
    quests_json = db.Column(db.Text, nullable=False)
    canva_embed_url = db.Column(db.String(500), nullable=True)
    {
  "name": "eco-learn-platform",
  "version": "1.0.0",
  "lockfileVersion": 3,
  "requires": true,
  "packages": {
    "": {
      "name": "eco-learn-platform",
      "version": "1.0.0",
      "license": "ISC",
      "dependencies": {
        "@google/genai": "^2.13.0",
        "cors": "^2.8.6",
        "dotenv": "^17.4.2",
        "express": "^5.2.1"
      },
      "devDependencies": {}
    },
    "node_modules/@google/genai": {
      "version": "2.13.0",
      "resolved": "https://registry.npmjs.org/@google/genai/-/genai-2.13.0.tgz",
      "integrity": "sha512-GM7C8Kaomvjz05x5JEO6+l3d/pciL9LxAG9dUjJLD7nTPZ9X0Cfsf2Z7eET6UjgWyUmxXCHtYnQoQ77F9+ZIOQ==",
      "hasInstallScript": true,
      "license": "Apache-2.0",
      "dependencies": {
        "google-auth-library": "^10.3.0",
        "p-retry": "^4.6.2",
        "protobufjs": "^7.5.4",
        "ws": "^8.18.0"
      },
      "engines": {
        "node": ">=20.0.0"
      },
      "peerDependencies": {
        "@modelcontextprotocol/sdk": "^1.25.2"
      },
      "peerDependenciesMeta": {
        "@modelcontextprotocol/sdk": {
          "optional": true
        }
      }
    },
    "node_modules/@protobufjs/aspromise": {
      "version": "1.1.2",
      "resolved": "https://registry.npmjs.org/@protobufjs/aspromise/-/aspromise-1.1.2.tgz",
      "integrity": "sha512-j+gKExEuLmKwvz3OgROXtrJ2UG2x8Ch2YZUxahh+s1F2HZ+wAceUNLkvy6zKCPVRkU++ZWQrdxsUeQXmcg4uoQ==",
      "license": "BSD-3-Clause"
    },
    "node_modules/@protobufjs/base64": {
      "version": "1.1.2",
      "resolved": "https://registry.npmjs.org/@protobufjs/base64/-/base64-1.1.2.tgz",
      "integrity": "sha512-AZkcAA5vnN/v4PDqKyMR5lx7hZttPDgClv83E//FMNhR2TMcLUhfRUBHCmSl0oi9zMgDDqRUJkSxO3wm85+XLg==",
      "license": "BSD-3-Clause"
    },
    "node_modules/@protobufjs/codegen": {
      "version": "2.0.5",
      "resolved": "https://registry.npmjs.org/@protobufjs/codegen/-/codegen-2.0.5.tgz",
      "integrity": "sha512-zgXFLzW3Ap33e6d0Wlj4MGIm6Ce8O89n/apUaGNB/jx+hw+ruWEp7EwGUshdLKVRCxZW12fp9r40E1mQrf/34g==",
      "license": "BSD-3-Clause"
    },
    "node_modules/@protobufjs/eventemitter": {
      "version": "1.1.1",
      "resolved": "https://registry.npmjs.org/@protobufjs/eventemitter/-/eventemitter-1.1.1.tgz",
      "integrity": "sha512-vW1GmwMZNnL+gMRaovlh9yZX74kc+TTU3FObkkurpMaRtBfLP3ldjS9KQWlwZgraRE0+dheEEoAxdzcJQ8eXZg==",
      "license": "BSD-3-Clause"
    },
    "node_modules/@protobufjs/fetch": {
      "version": "1.1.1",
      "resolved": "https://registry.npmjs.org/@protobufjs/fetch/-/fetch-1.1.1.tgz",
      "integrity": "sha512-GpptLrs57adMSuHi3VNj0mAF8dwh36LMaYF6XyJ6JMWlVsc+t42tm1HSEDmOs3A8fC9yyeisgLhsTVQokOZ0zw==",
      "license": "BSD-3-Clause",
      "dependencies": {
        "@protobufjs/aspromise": "^1.1.1"
      }
    },
    "node_modules/@protobufjs/float": {
      "version": "1.0.2",
      "resolved": "https://registry.npmjs.org/@protobufjs/float/-/float-1.0.2.tgz",
      "integrity": "sha512-Ddb+kVXlXst9d+R9PfTIxh1EdNkgoRe5tOX6t01f1lYWOvJnSPDBlG241QLzcyPdoNTsblLUdujGSE4RzrTZGQ==",
      "license": "BSD-3-Clause"
    },
    "node_modules/@protobufjs/path": {
      "version": "1.1.2",
      "resolved": "https://registry.npmjs.org/@protobufjs/path/-/path-1.1.2.tgz",
      "integrity": "sha512-6JOcJ5Tm08dOHAbdR3GrvP+yUUfkjG5ePsHYczMFLq3ZmMkAD98cDgcT2iA1lJ9NVwFd4tH/iSSoe44YWkltEA==",
      "license": "BSD-3-Clause"
    },
    "node_modules/@protobufjs/pool": {
      "version": "1.1.0",
      "resolved": "https://registry.npmjs.org/@protobufjs/pool/-/pool-1.1.0.tgz",
      "integrity": "sha512-0kELaGSIDBKvcgS4zkjz1PeddatrjYcmMWOlAuAPwAeccUrPHdUqo/J6LiymHHEiJT5NrF1UVwxY14f+fy4WQw==",
      "license": "BSD-3-Clause"
    },
    "node_modules/@protobufjs/utf8": {
      "version": "1.1.2",
      "resolved": "https://registry.npmjs.org/@protobufjs/utf8/-/utf8-1.1.2.tgz",
      "integrity": "sha512-b1UQwcEZ4yCnMCD8DAL1VlbvBJE9/IX4FTIp7BG1xYpf29SLazLSrqUkj4w7Y5y7cCVP6E5tcqqcI0xemPkHug==",
      "license": "BSD-3-Clause"
    },
    "node_modules/@types/node": {
      "version": "26.1.1",
      "resolved": "https://registry.npmjs.org/@types/node/-/node-26.1.1.tgz",
      "integrity": "sha512-nxAkRSVkN1Y0JC1W8ky/fTfkGsMmcrRsbx+3XoZE+rMOX71kLYTV7fLXpqud1GpbpP5TuffXFqfX7fH2GgZREw==",
      "license": "MIT",
      "dependencies": {
        "undici-types": "~8.3.0"
      }
    },
    "node_modules/@types/retry": {
      "version": "0.12.0",
      "resolved": "https://registry.npmjs.org/@types/retry/-/retry-0.12.0.tgz",
      "integrity": "sha512-wWKOClTTiizcZhXnPY4wikVAwmdYHp8q6DmC+EJUzAMsycb7HB32Kh9RN4+0gExjmPmZSAQjgURXIGATPegAvA==",
      "license": "MIT"
    },
    "node_modules/accepts": {
      "version": "2.0.0",
      "resolved": "https://registry.npmjs.org/accepts/-/accepts-2.0.0.tgz",
      "integrity": "sha512-5cvg6CtKwfgdmVqY1WIiXKc3Q1bkRqGLi+2W/6ao+6Y7gu/RCwRuAhGEzh5B4KlszSuTLgZYuqFqo5bImjNKng==",
      "license": "MIT",
      "dependencies": {
        "mime-types": "^3.0.0",
        "negotiator": "^1.0.0"
      },
      "engines": {
        "node": ">= 0.6"
      }
    },
    "node_modules/agent-base": {
      "version": "7.1.4",
      "resolved": "https://registry.npmjs.org/agent-base/-/agent-base-7.1.4.tgz",
      "integrity": "sha512-MnA+YT8fwfJPgBx3m60MNqakm30XOkyIoH1y6huTQvC0PwZG7ki8NacLBcrPbNoo8vEZy7Jpuk7+jMO+CUovTQ==",
      "license": "MIT",
      "engines": {
        "node": ">= 14"
      }
    },
    "node_modules/base64-js": {
      "version": "1.5.1",
      "resolved": "https://registry.npmjs.org/base64-js/-/base64-js-1.5.1.tgz",
      "integrity": "sha512-AKpaYlHn8t4SVbOHCy+b5+KKgvR4vrsD8vbvrbiQJps7fKDTkjkDry6ji0rUJjC0kzbNePLwzxq8iypo41qeWA==",
      "funding": [
        {
          "type": "github",
          "url": "https://github.com/sponsors/feross"
        },
        {
          "type": "patreon",
          "url": "https://www.patreon.com/feross"
        },
        {
          "type": "consulting",
          "url": "https://feross.org/support"
        }
      ],
      "license": "MIT"
    },
    "node_modules/bignumber.js": {
      "version": "9.3.1",
      "resolved": "https://registry.npmjs.org/bignumber.js/-/bignumber.js-9.3.1.tgz",
      "integrity": "sha512-Ko0uX15oIUS7wJ3Rb30Fs6SkVbLmPBAKdlm7q9+ak9bbIeFf0MwuBsQV6z7+X768/cHsfg+WlysDWJcmthjsjQ==",
      "license": "MIT",
      "engines": {
        "node": "*"
      }
    },
    "node_modules/body-parser": {
      "version": "2.3.0",
      "resolved": "https://registry.npmjs.org/body-parser/-/body-parser-2.3.0.tgz",
      "integrity": "sha512-2cGmJupaNgg+QUwVLAucDuWuoMZ6EX9iHDRswZ5lsNYEmwPaRknMPCLZz07yTzVq/83p4o/wzbDZbBrTvGGTIw==",
      "license": "MIT",
      "dependencies": {
        "bytes": "^3.1.2",
        "content-type": "^2.0.0",
        "debug": "^4.4.3",
        "http-errors": "^2.0.1",
        "iconv-lite": "^0.7.2",
        "on-finished": "^2.4.1",
        "qs": "^6.15.2",
        "raw-body": "^3.0.2",
        "type-is": "^2.1.0"
      },
      "engines": {
        "node": ">=18"
      },
      "funding": {
        "type": "opencollective",
        "url": "https://opencollective.com/express"
      }
    },
    "node_modules/body-parser/node_modules/content-type": {
      "version": "2.0.0",
      "resolved": "https://registry.npmjs.org/content-type/-/content-type-2.0.0.tgz",
      "integrity": "sha512-j/O/d7GcZCyNl7/hwZAb606rzqkyvaDctLmckbxLzHvFBzTJHuGEdodATcP3yIRoDrLHkIATJuvzbFlp/ki2cQ==",
      "license": "MIT",
      "engines": {
        "node": ">=18"
      },
      "funding": {
        "type": "opencollective",
        "url": "https://opencollective.com/express"
      }
    },
    "node_modules/buffer-equal-constant-time": {
      "version": "1.0.1",
      "resolved": "https://registry.npmjs.org/buffer-equal-constant-time/-/buffer-equal-constant-time-1.0.1.tgz",
      "integrity": "sha512-zRpUiDwd/xk6ADqPMATG8vc9VPrkck7T07OIx0gnjmJAnHnTVXNQG3vfvWNuiZIkwu9KrKdA1iJKfsfTVxE6NA==",
      "license": "BSD-3-Clause"
    },
    "node_modules/bytes": {
      "version": "3.1.2",
      "resolved": "https://registry.npmjs.org/bytes/-/bytes-3.1.2.tgz",
      "integrity": "sha512-/Nf7TyzTx6S3yRJObOAV7956r8cr2+Oj8AC5dt8wSP3BQAoeX58NoHyCU8P8zGkNXStjTSi6fzO6F0pBdcYbEg==",
      "license": "MIT",
      "engines": {
        "node": ">= 0.8"
      }
    },
    "node_modules/call-bind-apply-helpers": {
      "version": "1.0.2",
      "resolved": "https://registry.npmjs.org/call-bind-apply-helpers/-/call-bind-apply-helpers-1.0.2.tgz",
      "integrity": "sha512-Sp1ablJ0ivDkSzjcaJdxEunN5/XvksFJ2sMBFfq6x0ryhQV/2b/KwFe21cMpmHtPOSij8K99/wSfoEuTObmuMQ==",
      "license": "MIT",
      "dependencies": {
        "es-errors": "^1.3.0",
        "function-bind": "^1.1.2"
      },
      "engines": {
        "node": ">= 0.4"
      }
    },
    "node_modules/call-bound": {
      "version": "1.0.4",
      "resolved": "https://registry.npmjs.org/call-bound/-/call-bound-1.0.4.tgz",
      "integrity": "sha512-+ys997U96po4Kx/ABpBCqhA9EuxJaQWDQg7295H4hBphv3IZg0boBKuwYpt4YXp6MZ5AmZQnU/tyMTlRpaSejg==",
      "license": "MIT",
      "dependencies": {
        "call-bind-apply-helpers": "^1.0.2",
        "get-intrinsic": "^1.3.0"
      },
      "engines": {
        "node": ">= 0.4"
      },
      "funding": {
        "url": "https://github.com/sponsors/ljharb"
      }
    },
    "node_modules/content-disposition": {
      "version": "1.1.0",
      "resolved": "https://registry.npmjs.org/content-disposition/-/content-disposition-1.1.0.tgz",
      "integrity": "sha512-5jRCH9Z/+DRP7rkvY83B+yGIGX96OYdJmzngqnw2SBSxqCFPd0w2km3s5iawpGX8krnwSGmF0FW5Nhr0Hfai3g==",
      "license": "MIT",
      "engines": {
        "node": ">=18"
      },
      "funding": {
        "type": "opencollective",
        "url": "https://opencollective.com/express"
      }
    },
    "node_modules/content-type": {
      "version": "1.0.5",
      "resolved": "https://registry.npmjs.org/content-type/-/content-type-1.0.5.tgz",
      "integrity": "sha512-nTjqfcBFEipKdXCv4YDQWCfmcLZKm81ldF0pAopTvyrFGVbcR6P/VAAd5G7N+0tTr8QqiU0tFadD6FK4NtJwOA==",
      "license": "MIT",
      "engines": {
        "node": ">= 0.6"
      }
    },
    "node_modules/cookie": {
      "version": "0.7.2",
      "resolved": "https://registry.npmjs.org/cookie/-/cookie-0.7.2.tgz",
      "integrity": "sha512-yki5XnKuf750l50uGTllt6kKILY4nQ1eNIQatoXEByZ5dWgnKqbnqmTrBE5B4N7lrMJKQ2ytWMiTO2o0v6Ew/w==",
      "license": "MIT",
      "engines": {
        "node": ">= 0.6"
      }
    },
    "node_modules/cookie-signature": {
      "version": "1.2.2",
      "resolved": "https://registry.npmjs.org/cookie-signature/-/cookie-signature-1.2.2.tgz",
      "integrity": "sha512-D76uU73ulSXrD1UXF4KE2TMxVVwhsnCgfAyTg9k8P6KGZjlXKrOLe4dJQKI3Bxi5wjesZoFXJWElNWBjPZMbhg==",
      "license": "MIT",
      "engines": {
        "node": ">=6.6.0"
      }
    },
    "node_modules/cors": {
      "version": "2.8.6",
      "resolved": "https://registry.npmjs.org/cors/-/cors-2.8.6.tgz",
      "integrity": "sha512-tJtZBBHA6vjIAaF6EnIaq6laBBP9aq/Y3ouVJjEfoHbRBcHBAHYcMh/w8LDrk2PvIMMq8gmopa5D4V8RmbrxGw==",
      "license": "MIT",
      "dependencies": {
        "object-assign": "^4",
        "vary": "^1"
      },
      "engines": {
        "node": ">= 0.10"
      },
      "funding": {
        "type": "opencollective",
        "url": "https://opencollective.com/express"
      }
    },
    "node_modules/data-uri-to-buffer": {
      "version": "4.0.1",
      "resolved": "https://registry.npmjs.org/data-uri-to-buffer/-/data-uri-to-buffer-4.0.1.tgz",
      "integrity": "sha512-0R9ikRb668HB7QDxT1vkpuUBtqc53YyAwMwGeUFKRojY/NWKvdZ+9UYtRfGmhqNbRkTSVpMbmyhXipFFv2cb/A==",
      "license": "MIT",
      "engines": {
        "node": ">= 12"
      }
    },
    "node_modules/debug": {
      "version": "4.4.3",
      "resolved": "https://registry.npmjs.org/debug/-/debug-4.4.3.tgz",
      "integrity": "sha512-RGwwWnwQvkVfavKVt22FGLw+xYSdzARwm0ru6DhTVA3umU5hZc28V3kO4stgYryrTlLpuvgI9GiijltAjNbcqA==",
      "license": "MIT",
      "dependencies": {
        "ms": "^2.1.3"
      },
      "engines": {
        "node": ">=6.0"
      },
      "peerDependenciesMeta": {
        "supports-color": {
          "optional": true
        }
      }
    },
    "node_modules/depd": {
      "version": "2.0.0",
      "resolved": "https://registry.npmjs.org/depd/-/depd-2.0.0.tgz",
      "integrity": "sha512-g7nH6P6dyDioJogAAGprGpCtVImJhpPk/roCzdb3fIh61/s/nPsfR6onyMwkCAR/OlC3yBC0lESvUoQEAssIrw==",
      "license": "MIT",
      "engines": {
        "node": ">= 0.8"
      }
    },
    "node_modules/dotenv": {
      "version": "17.4.2",
      "resolved": "https://registry.npmjs.org/dotenv/-/dotenv-17.4.2.tgz",
      "integrity": "sha512-nI4U3TottKAcAD9LLud4Cb7b2QztQMUEfHbvhTH09bqXTxnSie8WnjPALV/WMCrJZ6UV/qHJ6L03OqO3LcdYZw==",
      "license": "BSD-2-Clause",
      "engines": {
        "node": ">=12"
      },
      "funding": {
        "url": "https://dotenvx.com"
      }
    },
    "node_modules/dunder-proto": {
      "version": "1.0.1",
      "resolved": "https://registry.npmjs.org/dunder-proto/-/dunder-proto-1.0.1.tgz",
      "integrity": "sha512-KIN/nDJBQRcXw0MLVhZE9iQHmG68qAVIBg9CqmUYjmQIhgij9U5MFvrqkUL5FbtyyzZuOeOt0zdeRe4UY7ct+A==",
      "license": "MIT",
      "dependencies": {
        "call-bind-apply-helpers": "^1.0.1",
        "es-errors": "^1.3.0",
        "gopd": "^1.2.0"
      },
      "engines": {
        "node": ">= 0.4"
      }
    },
    "node_modules/ecdsa-sig-formatter": {
      "version": "1.0.11",
      "resolved": "https://registry.npmjs.org/ecdsa-sig-formatter/-/ecdsa-sig-formatter-1.0.11.tgz",
      "integrity": "sha512-nagl3RYrbNv6kQkeJIpt6NJZy8twLB/2vtz6yN9Z4vRKHN4/QZJIEbqohALSgwKdnksuY3k5Addp5lg8sVoVcQ==",
      "license": "Apache-2.0",
      "dependencies": {
        "safe-buffer": "^5.0.1"
      }
    },
    "node_modules/ee-first": {
      "version": "1.1.1",
      "resolved": "https://registry.npmjs.org/ee-first/-/ee-first-1.1.1.tgz",
      "integrity": "sha512-WMwm9LhRUo+WUaRN+vRuETqG89IgZphVSNkdFgeb6sS/E4OrDIN7t48CAewSHXc6C8lefD8KKfr5vY61brQlow==",
      "license": "MIT"
    },
    "node_modules/encodeurl": {
      "version": "2.0.0",
      "resolved": "https://registry.npmjs.org/encodeurl/-/encodeurl-2.0.0.tgz",
      "integrity": "sha512-Q0n9HRi4m6JuGIV1eFlmvJB7ZEVxu93IrMyiMsGC0lrMJMWzRgx6WGquyfQgZVb31vhGgXnfmPNNXmxnOkRBrg==",
      "license": "MIT",
      "engines": {
        "node": ">= 0.8"
      }
    },
    "node_modules/es-define-property": {
      "version": "1.0.1",
      "resolved": "https://registry.npmjs.org/es-define-property/-/es-define-property-1.0.1.tgz",
      "integrity": "sha512-e3nRfgfUZ4rNGL232gUgX06QNyyez04KdjFrF+LTRoOXmrOgFKDg4BCdsjW8EnT69eqdYGmRpJwiPVYNrCaW3g==",
      "license": "MIT",
      "engines": {
        "node": ">= 0.4"
      }
    },
    "node_modules/es-errors": {
      "version": "1.3.0",
      "resolved": "https://registry.npmjs.org/es-errors/-/es-errors-1.3.0.tgz",
      "integrity": "sha512-Zf5H2Kxt2xjTvbJvP2ZWLEICxA6j+hAmMzIlypy4xcBg1vKVnx89Wy0GbS+kf5cwCVFFzdCFh2XSCFNULS6csw==",
      "license": "MIT",
      "engines": {
        "node": ">= 0.4"
      }
    },
    "node_modules/es-object-atoms": {
      "version": "1.1.2",
      "resolved": "https://registry.npmjs.org/es-object-atoms/-/es-object-atoms-1.1.2.tgz",
      "integrity": "sha512-HWcBoN6NileqtSydK2FqHbS/LoDd2pqrnQHLyJzBj4kOp/ky2MWMN694xOfkK8/SnUsW2DH7EfyVlydKCsm1Zw==",
      "license": "MIT",
      "dependencies": {
        "es-errors": "^1.3.0"
      },
      "engines": {
        "node": ">= 0.4"
      }
    },
    "node_modules/escape-html": {
      "version": "1.0.3",
      "resolved": "https://registry.npmjs.org/escape-html/-/escape-html-1.0.3.tgz",
      "integrity": "sha512-NiSupZ4OeuGwr68lGIeym/ksIZMJodUGOSCZ/FSnTxcrekbvqrgdUxlJOMpijaKZVjAJrWrGs/6Jy8OMuyj9ow==",
      "license": "MIT"
    },
    "node_modules/etag": {
      "version": "1.8.1",
      "resolved": "https://registry.npmjs.org/etag/-/etag-1.8.1.tgz",
      "integrity": "sha512-aIL5Fx7mawVa300al2BnEE4iNvo1qETxLrPI/o05L7z6go7fCw1J6EQmbK4FmJ2AS7kgVF/KEZWufBfdClMcPg==",
      "license": "MIT",
      "engines": {
        "node": ">= 0.6"
      }
    },
    "node_modules/express": {
      "version": "5.2.1",
      "resolved": "https://registry.npmjs.org/express/-/express-5.2.1.tgz",
      "integrity": "sha512-hIS4idWWai69NezIdRt2xFVofaF4j+6INOpJlVOLDO8zXGpUVEVzIYk12UUi2JzjEzWL3IOAxcTubgz9Po0yXw==",
      "license": "MIT",
      "dependencies": {
        "accepts": "^2.0.0",
        "body-parser": "^2.2.1",
        "content-disposition": "^1.0.0",
        "content-type": "^1.0.5",
        "cookie": "^0.7.1",
        "cookie-signature": "^1.2.1",
        "debug": "^4.4.0",
        "depd": "^2.0.0",
        "encodeurl": "^2.0.0",
        "escape-html": "^1.0.3",
        "etag": "^1.8.1",
        "finalhandler": "^2.1.0",
        "fresh": "^2.0.0",
        "http-errors": "^2.0.0",
        "merge-descriptors": "^2.0.0",
        "mime-types": "^3.0.0",
        "on-finished": "^2.4.1",
        "once": "^1.4.0",
        "parseurl": "^1.3.3",
        "proxy-addr": "^2.0.7",
        "qs": "^6.14.0",
        "range-parser": "^1.2.1",
        "router": "^2.2.0",
        "send": "^1.1.0",
        "serve-static": "^2.2.0",
        "statuses": "^2.0.1",
        "type-is": "^2.0.1",
        "vary": "^1.1.2"
      },
      "engines": {
        "node": ">= 18"
      },
      "funding": {
        "type": "opencollective",
        "url": "https://opencollective.com/express"
      }
    },
    "node_modules/extend": {
      "version": "3.0.2",
      "resolved": "https://registry.npmjs.org/extend/-/extend-3.0.2.tgz",
      "integrity": "sha512-fjquC59cD7CyW6urNXK0FBufkZcoiGG80wTuPujX590cB5Ttln20E2UB4S/WARVqhXffZl2LNgS+gQdPIIim/g==",
      "license": "MIT"
    },
    "node_modules/fetch-blob": {
      "version": "3.2.0",
      "resolved": "https://registry.npmjs.org/fetch-blob/-/fetch-blob-3.2.0.tgz",
      "integrity": "sha512-7yAQpD2UMJzLi1Dqv7qFYnPbaPx7ZfFK6PiIxQ4PfkGPyNyl2Ugx+a/umUonmKqjhM4DnfbMvdX6otXq83soQQ==",
      "funding": [
        {
          "type": "github",
          "url": "https://github.com/sponsors/jimmywarting"
        },
        {
          "type": "paypal",
          "url": "https://paypal.me/jimmywarting"
        }
      ],
      "license": "MIT",
      "dependencies": {
        "node-domexception": "^1.0.0",
        "web-streams-polyfill": "^3.0.3"
      },
      "engines": {
        "node": "^12.20 || >= 14.13"
      }
    },
    "node_modules/finalhandler": {
      "version": "2.1.1",
      "resolved": "https://registry.npmjs.org/finalhandler/-/finalhandler-2.1.1.tgz",
      "integrity": "sha512-S8KoZgRZN+a5rNwqTxlZZePjT/4cnm0ROV70LedRHZ0p8u9fRID0hJUZQpkKLzro8LfmC8sx23bY6tVNxv8pQA==",
      "license": "MIT",
      "dependencies": {
        "debug": "^4.4.0",
        "encodeurl": "^2.0.0",
        "escape-html": "^1.0.3",
        "on-finished": "^2.4.1",
        "parseurl": "^1.3.3",
        "statuses": "^2.0.1"
      },
      "engines": {
        "node": ">= 18.0.0"
      },
      "funding": {
        "type": "opencollective",
        "url": "https://opencollective.com/express"
      }
    },
    "node_modules/formdata-polyfill": {
      "version": "4.0.10",
      "resolved": "https://registry.npmjs.org/formdata-polyfill/-/formdata-polyfill-4.0.10.tgz",
      "integrity": "sha512-buewHzMvYL29jdeQTVILecSaZKnt/RJWjoZCF5OW60Z67/GmSLBkOFM7qh1PI3zFNtJbaZL5eQu1vLfazOwj4g==",
      "license": "MIT",
      "dependencies": {
        "fetch-blob": "^3.1.2"
      },
      "engines": {
        "node": ">=12.20.0"
      }
    },
    "node_modules/forwarded": {
      "version": "0.2.0",
      "resolved": "https://registry.npmjs.org/forwarded/-/forwarded-0.2.0.tgz",
      "integrity": "sha512-buRG0fpBtRHSTCOASe6hD258tEubFoRLb4ZNA6NxMVHNw2gOcwHo9wyablzMzOA5z9xA9L1KNjk/Nt6MT9aYow==",
      "license": "MIT",
      "engines": {
        "node": ">= 0.6"
      }
    },
    "node_modules/fresh": {
      "version": "2.0.0",
      "resolved": "https://registry.npmjs.org/fresh/-/fresh-2.0.0.tgz",
      "integrity": "sha512-Rx/WycZ60HOaqLKAi6cHRKKI7zxWbJ31MhntmtwMoaTeF7XFH9hhBp8vITaMidfljRQ6eYWCKkaTK+ykVJHP2A==",
      "license": "MIT",
      "engines": {
        "node": ">= 0.8"
      }
    },
    "node_modules/function-bind": {
      "version": "1.1.2",
      "resolved": "https://registry.npmjs.org/function-bind/-/function-bind-1.1.2.tgz",
      "integrity": "sha512-7XHNxH7qX9xG5mIwxkhumTox/MIRNcOgDrxWsMt2pAr23WHp6MrRlN7FBSFpCpr+oVO0F744iUgR82nJMfG2SA==",
      "license": "MIT",
      "funding": {
        "url": "https://github.com/sponsors/ljharb"
      }
    },
    "node_modules/gaxios": {
      "version": "7.2.0",
      "resolved": "https://registry.npmjs.org/gaxios/-/gaxios-7.2.0.tgz",
      "integrity": "sha512-CUVb4wcYe+771XevyH6HtGmXFAGGKkIC3kswAP8Z1JCe0j80JMaTPZH930DWFrvo0atjh18Arc0pEyUCWa5bfg==",
      "license": "Apache-2.0",
      "dependencies": {
        "extend": "^3.0.2",
        "https-proxy-agent": "^7.0.1",
        "node-fetch": "^3.3.2"
      },
      "engines": {
        "node": ">=18"
      }
    },
    "node_modules/gcp-metadata": {
      "version": "8.1.2",
      "resolved": "https://registry.npmjs.org/gcp-metadata/-/gcp-metadata-8.1.2.tgz",
      "integrity": "sha512-zV/5HKTfCeKWnxG0Dmrw51hEWFGfcF2xiXqcA3+J90WDuP0SvoiSO5ORvcBsifmx/FoIjgQN3oNOGaQ5PhLFkg==",
      "license": "Apache-2.0",
      "dependencies": {
        "gaxios": "^7.0.0",
        "google-logging-utils": "^1.0.0",
        "json-bigint": "^1.0.0"
      },
      "engines": {
        "node": ">=18"
      }
    },
    "node_modules/get-intrinsic": {
      "version": "1.3.0",
      "resolved": "https://registry.npmjs.org/get-intrinsic/-/get-intrinsic-1.3.0.tgz",
      "integrity": "sha512-9fSjSaos/fRIVIp+xSJlE6lfwhES7LNtKaCBIamHsjr2na1BiABJPo0mOjjz8GJDURarmCPGqaiVg5mfjb98CQ==",
      "license": "MIT",
      "dependencies": {
        "call-bind-apply-helpers": "^1.0.2",
        "es-define-property": "^1.0.1",
        "es-errors": "^1.3.0",
        "es-object-atoms": "^1.1.1",
        "function-bind": "^1.1.2",
        "get-proto": "^1.0.1",
        "gopd": "^1.2.0",
        "has-symbols": "^1.1.0",
        "hasown": "^2.0.2",
        "math-intrinsics": "^1.1.0"
      },
      "engines": {
        "node": ">= 0.4"
      },
      "funding": {
        "url": "https://github.com/sponsors/ljharb"
      }
    },
    "node_modules/get-proto": {
      "version": "1.0.1",
      "resolved": "https://registry.npmjs.org/get-proto/-/get-proto-1.0.1.tgz",
      "integrity": "sha512-sTSfBjoXBp89JvIKIefqw7U2CCebsc74kiY6awiGogKtoSGbgjYE/G/+l9sF3MWFPNc9IcoOC4ODfKHfxFmp0g==",
      "license": "MIT",
      "dependencies": {
        "dunder-proto": "^1.0.1",
        "es-object-atoms": "^1.0.0"
      },
      "engines": {
        "node": ">= 0.4"
      }
    },
    "node_modules/google-auth-library": {
      "version": "10.9.0",
      "resolved": "https://registry.npmjs.org/google-auth-library/-/google-auth-library-10.9.0.tgz",
      "integrity": "sha512-xtvUqvINPhTaBm7nXqlYPcrMHJPm1lCNdSovxnKKhTm+4JsvQ+KGVYJViLoH9Yxu8w+T0Qv5HubzYT9BLrppJg==",
      "license": "Apache-2.0",
      "dependencies": {
        "base64-js": "^1.3.0",
        "ecdsa-sig-formatter": "^1.0.11",
        "gaxios": "^7.1.4",
        "gcp-metadata": "8.1.2",
        "google-logging-utils": "1.1.3",
        "jws": "^4.0.0"
      },
      "engines": {
        "node": ">=18"
      }
    },
    "node_modules/google-logging-utils": {
      "version": "1.1.3",
      "resolved": "https://registry.npmjs.org/google-logging-utils/-/google-logging-utils-1.1.3.tgz",
      "integrity": "sha512-eAmLkjDjAFCVXg7A1unxHsLf961m6y17QFqXqAXGj/gVkKFrEICfStRfwUlGNfeCEjNRa32JEWOUTlYXPyyKvA==",
      "license": "Apache-2.0",
      "engines": {
        "node": ">=14"
      }
    },
    "node_modules/gopd": {
      "version": "1.2.0",
      "resolved": "https://registry.npmjs.org/gopd/-/gopd-1.2.0.tgz",
      "integrity": "sha512-ZUKRh6/kUFoAiTAtTYPZJ3hw9wNxx+BIBOijnlG9PnrJsCcSjs1wyyD6vJpaYtgnzDrKYRSqf3OO6Rfa93xsRg==",
      "license": "MIT",
      "engines": {
        "node": ">= 0.4"
      },
      "funding": {
        "url": "https://github.com/sponsors/ljharb"
      }
    },
    "node_modules/has-symbols": {
      "version": "1.1.0",
      "resolved": "https://registry.npmjs.org/has-symbols/-/has-symbols-1.1.0.tgz",
      "integrity": "sha512-1cDNdwJ2Jaohmb3sg4OmKaMBwuC48sYni5HUw2DvsC8LjGTLK9h+eb1X6RyuOHe4hT0ULCW68iomhjUoKUqlPQ==",
      "license": "MIT",
      "engines": {
        "node": ">= 0.4"
      },
      "funding": {
        "url": "https://github.com/sponsors/ljharb"
      }
    },
    "node_modules/hasown": {
      "version": "2.0.4",
      "resolved": "https://registry.npmjs.org/hasown/-/hasown-2.0.4.tgz",
      "integrity": "sha512-T2UbfbBEF32wiepXIsMlTW9+dDYC6wMh/t/vYA4tuOMKqWz/n3vr1NFSxQiyP+zk2mXsoMA/i/7qV6LKut1t1A==",
      "license": "MIT",
      "dependencies": {
        "function-bind": "^1.1.2"
      },
      "engines": {
        "node": ">= 0.4"
      }
    },
    "node_modules/http-errors": {
      "version": "2.0.1",
      "resolved": "https://registry.npmjs.org/http-errors/-/http-errors-2.0.1.tgz",
      "integrity": "sha512-4FbRdAX+bSdmo4AUFuS0WNiPz8NgFt+r8ThgNWmlrjQjt1Q7ZR9+zTlce2859x4KSXrwIsaeTqDoKQmtP8pLmQ==",
      "license": "MIT",
      "dependencies": {
        "depd": "~2.0.0",
        "inherits": "~2.0.4",
        "setprototypeof": "~1.2.0",
        "statuses": "~2.0.2",
        "toidentifier": "~1.0.1"
      },
      "engines": {
        "node": ">= 0.8"
      },
      "funding": {
        "type": "opencollective",
        "url": "https://opencollective.com/express"
      }
    },
    "node_modules/https-proxy-agent": {
      "version": "7.0.6",
      "resolved": "https://registry.npmjs.org/https-proxy-agent/-/https-proxy-agent-7.0.6.tgz",
      "integrity": "sha512-vK9P5/iUfdl95AI+JVyUuIcVtd4ofvtrOr3HNtM2yxC9bnMbEdp3x01OhQNnjb8IJYi38VlTE3mBXwcfvywuSw==",
      "license": "MIT",
      "dependencies": {
        "agent-base": "^7.1.2",
        "debug": "4"
      },
      "engines": {
        "node": ">= 14"
      }
    },
    "node_modules/iconv-lite": {
      "version": "0.7.3",
      "resolved": "https://registry.npmjs.org/iconv-lite/-/iconv-lite-0.7.3.tgz",
      "integrity": "sha512-IKXpvIzjnC9XTAUbVBcMfGS0EPaIXtW6v+zr+RRp+hqULEpo0owZax6wyRwPOJbWbzjYspQwusTsfVr0ifh4uQ==",
      "license": "MIT",
      "dependencies": {
        "safer-buffer": ">= 2.1.2 < 3.0.0"
      },
      "engines": {
        "node": ">=0.10.0"
      },
      "funding": {
        "type": "opencollective",
        "url": "https://opencollective.com/express"
      }
    },
    "node_modules/inherits": {
      "version": "2.0.4",
      "resolved": "https://registry.npmjs.org/inherits/-/inherits-2.0.4.tgz",
      "integrity": "sha512-k/vGaX4/Yla3WzyMCvTQOXYeIHvqOKtnqBduzTHpzpQZzAskKMhZ2K+EnBiSM9zGSoIFeMpXKxa4dYeZIQqewQ==",
      "license": "ISC"
    },
    "node_modules/ipaddr.js": {
      "version": "1.9.1",
      "resolved": "https://registry.npmjs.org/ipaddr.js/-/ipaddr.js-1.9.1.tgz",
      "integrity": "sha512-0KI/607xoxSToH7GjN1FfSbLoU0+btTicjsQSWQlh/hZykN8KpmMf7uYwPW3R+akZ6R/w18ZlXSHBYXiYUPO3g==",
      "license": "MIT",
      "engines": {
        "node": ">= 0.10"
      }
    },
    "node_modules/is-promise": {
      "version": "4.0.0",
      "resolved": "https://registry.npmjs.org/is-promise/-/is-promise-4.0.0.tgz",
      "integrity": "sha512-hvpoI6korhJMnej285dSg6nu1+e6uxs7zG3BYAm5byqDsgJNWwxzM6z6iZiAgQR4TJ30JmBTOwqZUw3WlyH3AQ==",
      "license": "MIT"
    },
    "node_modules/json-bigint": {
      "version": "1.0.0",
      "resolved": "https://registry.npmjs.org/json-bigint/-/json-bigint-1.0.0.tgz",
      "integrity": "sha512-SiPv/8VpZuWbvLSMtTDU8hEfrZWg/mH/nV/b4o0CYbSxu1UIQPLdwKOCIyLQX+VIPO5vrLX3i8qtqFyhdPSUSQ==",
      "license": "MIT",
      "dependencies": {
        "bignumber.js": "^9.0.0"
      }
    },
    "node_modules/jwa": {
      "version": "2.0.1",
      "resolved": "https://registry.npmjs.org/jwa/-/jwa-2.0.1.tgz",
      "integrity": "sha512-hRF04fqJIP8Abbkq5NKGN0Bbr3JxlQ+qhZufXVr0DvujKy93ZCbXZMHDL4EOtodSbCWxOqR8MS1tXA5hwqCXDg==",
      "license": "MIT",
      "dependencies": {
        "buffer-equal-constant-time": "^1.0.1",
        "ecdsa-sig-formatter": "1.0.11",
        "safe-buffer": "^5.0.1"
      }
    },
    "node_modules/jws": {
      "version": "4.0.1",
      "resolved": "https://registry.npmjs.org/jws/-/jws-4.0.1.tgz",
      "integrity": "sha512-EKI/M/yqPncGUUh44xz0PxSidXFr/+r0pA70+gIYhjv+et7yxM+s29Y+VGDkovRofQem0fs7Uvf4+YmAdyRduA==",
      "license": "MIT",
      "dependencies": {
        "jwa": "^2.0.1",
        "safe-buffer": "^5.0.1"
      }
    },
    "node_modules/long": {
      "version": "5.3.2",
      "resolved": "https://registry.npmjs.org/long/-/long-5.3.2.tgz",
      "integrity": "sha512-mNAgZ1GmyNhD7AuqnTG3/VQ26o760+ZYBPKjPvugO8+nLbYfX6TVpJPseBvopbdY+qpZ/lKUnmEc1LeZYS3QAA==",
      "license": "Apache-2.0"
    },
    "node_modules/math-intrinsics": {
      "version": "1.1.0",
      "resolved": "https://registry.npmjs.org/math-intrinsics/-/math-intrinsics-1.1.0.tgz",
      "integrity": "sha512-/IXtbwEk5HTPyEwyKX6hGkYXxM9nbj64B+ilVJnC/R6B0pH5G4V3b0pVbL7DBj4tkhBAppbQUlf6F6Xl9LHu1g==",
      "license": "MIT",
      "engines": {
        "node": ">= 0.4"
      }
    },
    "node_modules/media-typer": {
      "version": "1.1.0",
      "resolved": "https://registry.npmjs.org/media-typer/-/media-typer-1.1.0.tgz",
      "integrity": "sha512-aisnrDP4GNe06UcKFnV5bfMNPBUw4jsLGaWwWfnH3v02GnBuXX2MCVn5RbrWo0j3pczUilYblq7fQ7Nw2t5XKw==",
      "license": "MIT",
      "engines": {
        "node": ">= 0.8"
      }
    },
    "node_modules/merge-descriptors": {
      "version": "2.0.0",
      "resolved": "https://registry.npmjs.org/merge-descriptors/-/merge-descriptors-2.0.0.tgz",
      "integrity": "sha512-Snk314V5ayFLhp3fkUREub6WtjBfPdCPY1Ln8/8munuLuiYhsABgBVWsozAG+MWMbVEvcdcpbi9R7ww22l9Q3g==",
      "license": "MIT",
      "engines": {
        "node": ">=18"
      },
      "funding": {
        "url": "https://github.com/sponsors/sindresorhus"
      }
    },
    "node_modules/mime-db": {
      "version": "1.54.0",
      "resolved": "https://registry.npmjs.org/mime-db/-/mime-db-1.54.0.tgz",
      "integrity": "sha512-aU5EJuIN2WDemCcAp2vFBfp/m4EAhWJnUNSSw0ixs7/kXbd6Pg64EmwJkNdFhB8aWt1sH2CTXrLxo/iAGV3oPQ==",
      "license": "MIT",
      "engines": {
        "node": ">= 0.6"
      }
    },
    "node_modules/mime-types": {
      "version": "3.0.2",
      "resolved": "https://registry.npmjs.org/mime-types/-/mime-types-3.0.2.tgz",
      "integrity": "sha512-Lbgzdk0h4juoQ9fCKXW4by0UJqj+nOOrI9MJ1sSj4nI8aI2eo1qmvQEie4VD1glsS250n15LsWsYtCugiStS5A==",
      "license": "MIT",
      "dependencies": {
        "mime-db": "^1.54.0"
      },
      "engines": {
        "node": ">=18"
      },
      "funding": {
        "type": "opencollective",
        "url": "https://opencollective.com/express"
      }
    },
    "node_modules/ms": {
      "version": "2.1.3",
      "resolved": "https://registry.npmjs.org/ms/-/ms-2.1.3.tgz",
      "integrity": "sha512-6FlzubTLZG3J2a/NVCAleEhjzq5oxgHyaCU9yYXvcLsvoVaHJq/s5xXI6/XXP6tz7R9xAOtHnSO/tXtF3WRTlA==",
      "license": "MIT"
    },
    "node_modules/negotiator": {
      "version": "1.0.0",
      "resolved": "https://registry.npmjs.org/negotiator/-/negotiator-1.0.0.tgz",
      "integrity": "sha512-8Ofs/AUQh8MaEcrlq5xOX0CQ9ypTF5dl78mjlMNfOK08fzpgTHQRQPBxcPlEtIw0yRpws+Zo/3r+5WRby7u3Gg==",
      "license": "MIT",
      "engines": {
        "node": ">= 0.6"
      }
    },
    "node_modules/node-domexception": {
      "version": "1.0.0",
      "resolved": "https://registry.npmjs.org/node-domexception/-/node-domexception-1.0.0.tgz",
      "integrity": "sha512-/jKZoMpw0F8GRwl4/eLROPA3cfcXtLApP0QzLmUT/HuPCZWyB7IY9ZrMeKw2O/nFIqPQB3PVM9aYm0F312AXDQ==",
      "deprecated": "Use your platform's native DOMException instead",
      "funding": [
        {
          "type": "github",
          "url": "https://github.com/sponsors/jimmywarting"
        },
        {
          "type": "github",
          "url": "https://paypal.me/jimmywarting"
        }
      ],
      "license": "MIT",
      "engines": {
        "node": ">=10.5.0"
      }
    },
    "node_modules/node-fetch": {
      "version": "3.3.2",
      "resolved": "https://registry.npmjs.org/node-fetch/-/node-fetch-3.3.2.tgz",
      "integrity": "sha512-dRB78srN/l6gqWulah9SrxeYnxeddIG30+GOqK/9OlLVyLg3HPnr6SqOWTWOXKRwC2eGYCkZ59NNuSgvSrpgOA==",
      "license": "MIT",
      "dependencies": {
        "data-uri-to-buffer": "^4.0.0",
        "fetch-blob": "^3.1.4",
        "formdata-polyfill": "^4.0.10"
      },
      "engines": {
        "node": "^12.20.0 || ^14.13.1 || >=16.0.0"
      },
      "funding": {
        "type": "opencollective",
        "url": "https://opencollective.com/node-fetch"
      }
    },
    "node_modules/object-assign": {
      "version": "4.1.1",
      "resolved": "https://registry.npmjs.org/object-assign/-/object-assign-4.1.1.tgz",
      "integrity": "sha512-rJgTQnkUnH1sFw8yT6VSU3zD3sWmu6sZhIseY8VX+GRu3P6F7Fu+JNDoXfklElbLJSnc3FUQHVe4cU5hj+BcUg==",
      "license": "MIT",
      "engines": {
        "node": ">=0.10.0"
      }
    },
    "node_modules/object-inspect": {
      "version": "1.13.4",
      "resolved": "https://registry.npmjs.org/object-inspect/-/object-inspect-1.13.4.tgz",
      "integrity": "sha512-W67iLl4J2EXEGTbfeHCffrjDfitvLANg0UlX3wFUUSTx92KXRFegMHUVgSqE+wvhAbi4WqjGg9czysTV2Epbew==",
      "license": "MIT",
      "engines": {
        "node": ">= 0.4"
      },
      "funding": {
        "url": "https://github.com/sponsors/ljharb"
      }
    },
    "node_modules/on-finished": {
      "version": "2.4.1",
      "resolved": "https://registry.npmjs.org/on-finished/-/on-finished-2.4.1.tgz",
      "integrity": "sha512-oVlzkg3ENAhCk2zdv7IJwd/QUD4z2RxRwpkcGY8psCVcCYZNq4wYnVWALHM+brtuJjePWiYF/ClmuDr8Ch5+kg==",
      "license": "MIT",
      "dependencies": {
        "ee-first": "1.1.1"
      },
      "engines": {
        "node": ">= 0.8"
      }
    },
    "node_modules/once": {
      "version": "1.4.0",
      "resolved": "https://registry.npmjs.org/once/-/once-1.4.0.tgz",
      "integrity": "sha512-lNaJgI+2Q5URQBkccEKHTQOPaXdUxnZZElQTZY0MFUAuaEqe1E+Nyvgdz/aIyNi6Z9MzO5dv1H8n58/GELp3+w==",
      "license": "ISC",
      "dependencies": {
        "wrappy": "1"
      }
    },
    "node_modules/p-retry": {
      "version": "4.6.2",
      "resolved": "https://registry.npmjs.org/p-retry/-/p-retry-4.6.2.tgz",
      "integrity": "sha512-312Id396EbJdvRONlngUx0NydfrIQ5lsYu0znKVUzVvArzEIt08V1qhtyESbGVd1FGX7UKtiFp5uwKZdM8wIuQ==",
      "license": "MIT",
      "dependencies": {
        "@types/retry": "0.12.0",
        "retry": "^0.13.1"
      },
      "engines": {
        "node": ">=8"
      }
    },
    "node_modules/parseurl": {
      "version": "1.3.3",
      "resolved": "https://registry.npmjs.org/parseurl/-/parseurl-1.3.3.tgz",
      "integrity": "sha512-CiyeOxFT/JZyN5m0z9PfXw4SCBJ6Sygz1Dpl0wqjlhDEGGBP1GnsUVEL0p63hoG1fcj3fHynXi9NYO4nWOL+qQ==",
      "license": "MIT",
      "engines": {
        "node": ">= 0.8"
      }
    },
    "node_modules/path-to-regexp": {
      "version": "8.4.2",
      "resolved": "https://registry.npmjs.org/path-to-regexp/-/path-to-regexp-8.4.2.tgz",
      "integrity": "sha512-qRcuIdP69NPm4qbACK+aDogI5CBDMi1jKe0ry5rSQJz8JVLsC7jV8XpiJjGRLLol3N+R5ihGYcrPLTno6pAdBA==",
      "license": "MIT",
      "funding": {
        "type": "opencollective",
        "url": "https://opencollective.com/express"
      }
    },
    "node_modules/protobufjs": {
      "version": "7.6.5",
      "resolved": "https://registry.npmjs.org/protobufjs/-/protobufjs-7.6.5.tgz",
      "integrity": "sha512-/FPD0nUc9jH6rfFjji9IBqOz4pcSE3CsT1m7Ep6Mdb0LxSUMj8hgl6GomOvZzpNpAqqGaXA0P3VSrZLFzIhQrw==",
      "hasInstallScript": true,
      "license": "BSD-3-Clause",
      "dependencies": {
        "@protobufjs/aspromise": "^1.1.2",
        "@protobufjs/base64": "^1.1.2",
        "@protobufjs/codegen": "^2.0.5",
        "@protobufjs/eventemitter": "^1.1.1",
        "@protobufjs/fetch": "^1.1.1",
        "@protobufjs/float": "^1.0.2",
        "@protobufjs/path": "^1.1.2",
        "@protobufjs/pool": "^1.1.0",
        "@protobufjs/utf8": "^1.1.1",
        "@types/node": ">=13.7.0",
        "long": "^5.3.2"
      },
      "engines": {
        "node": ">=12.0.0"
      }
    },
    "node_modules/proxy-addr": {
      "version": "2.0.7",
      "resolved": "https://registry.npmjs.org/proxy-addr/-/proxy-addr-2.0.7.tgz",
      "integrity": "sha512-llQsMLSUDUPT44jdrU/O37qlnifitDP+ZwrmmZcoSKyLKvtZxpyV0n2/bD/N4tBAAZ/gJEdZU7KMraoK1+XYAg==",
      "license": "MIT",
      "dependencies": {
        "forwarded": "0.2.0",
        "ipaddr.js": "1.9.1"
      },
      "engines": {
        "node": ">= 0.10"
      }
    },
    "node_modules/qs": {
      "version": "6.15.3",
      "resolved": "https://registry.npmjs.org/qs/-/qs-6.15.3.tgz",
      "integrity": "sha512-O9gl3zCl5h5blw1KGUzQKhA5oUXSl8rwUIM5o0S3nCXMliSvy5Dzx7/DJcI+SwgICv+IneSZwhBh1oSyEHA71A==",
      "license": "BSD-3-Clause",
      "dependencies": {
        "es-define-property": "^1.0.1",
        "side-channel": "^1.1.1"
      },
      "engines": {
        "node": ">=0.6"
      },
      "funding": {
        "url": "https://github.com/sponsors/ljharb"
      }
    },
    "node_modules/range-parser": {
      "version": "1.3.0",
      "resolved": "https://registry.npmjs.org/range-parser/-/range-parser-1.3.0.tgz",
      "integrity": "sha512-hek2mFQpPuI4E1BBKrSto+BU3e3x4xuarsbiwr3+lf7p44juvFMV0XFWQAP3xUyqXA4RrXLIoaSUGbSt056ZMw==",
      "license": "MIT",
      "engines": {
        "node": ">= 0.6"
      },
      "funding": {
        "type": "opencollective",
        "url": "https://opencollective.com/express"
      }
    },
    "node_modules/raw-body": {
      "version": "3.0.2",
      "resolved": "https://registry.npmjs.org/raw-body/-/raw-body-3.0.2.tgz",
      "integrity": "sha512-K5zQjDllxWkf7Z5xJdV0/B0WTNqx6vxG70zJE4N0kBs4LovmEYWJzQGxC9bS9RAKu3bgM40lrd5zoLJ12MQ5BA==",
      "license": "MIT",
      "dependencies": {
        "bytes": "~3.1.2",
        "http-errors": "~2.0.1",
        "iconv-lite": "~0.7.0",
        "unpipe": "~1.0.0"
      },
      "engines": {
        "node": ">= 0.10"
      }
    },
    "node_modules/retry": {
      "version": "0.13.1",
      "resolved": "https://registry.npmjs.org/retry/-/retry-0.13.1.tgz",
      "integrity": "sha512-XQBQ3I8W1Cge0Seh+6gjj03LbmRFWuoszgK9ooCpwYIrhhoO80pfq4cUkU5DkknwfOfFteRwlZ56PYOGYyFWdg==",
      "license": "MIT",
      "engines": {
        "node": ">= 4"
      }
    },
    "node_modules/router": {
      "version": "2.2.0",
      "resolved": "https://registry.npmjs.org/router/-/router-2.2.0.tgz",
      "integrity": "sha512-nLTrUKm2UyiL7rlhapu/Zl45FwNgkZGaCpZbIHajDYgwlJCOzLSk+cIPAnsEqV955GjILJnKbdQC1nVPz+gAYQ==",
      "license": "MIT",
      "dependencies": {
        "debug": "^4.4.0",
        "depd": "^2.0.0",
        "is-promise": "^4.0.0",
        "parseurl": "^1.3.3",
        "path-to-regexp": "^8.0.0"
      },
      "engines": {
        "node": ">= 18"
      }
    },
    "node_modules/safe-buffer": {
      "version": "5.2.1",
      "resolved": "https://registry.npmjs.org/safe-buffer/-/safe-buffer-5.2.1.tgz",
      "integrity": "sha512-rp3So07KcdmmKbGvgaNxQSJr7bGVSVk5S9Eq1F+ppbRo70+YeaDxkw5Dd8NPN+GD6bjnYm2VuPuCXmpuYvmCXQ==",
      "funding": [
        {
          "type": "github",
          "url": "https://github.com/sponsors/feross"
        },
        {
          "type": "patreon",
          "url": "https://www.patreon.com/feross"
        },
        {
          "type": "consulting",
          "url": "https://feross.org/support"
        }
      ],
      "license": "MIT"
    },
    "node_modules/safer-buffer": {
      "version": "2.1.2",
      "resolved": "https://registry.npmjs.org/safer-buffer/-/safer-buffer-2.1.2.tgz",
      "integrity": "sha512-YZo3K82SD7Riyi0E1EQPojLz7kpepnSQI9IyPbHHg1XXXevb5dJI7tpyN2ADxGcQbHG7vcyRHk0cbwqcQriUtg==",
      "license": "MIT"
    },
    "node_modules/send": {
      "version": "1.2.1",
      "resolved": "https://registry.npmjs.org/send/-/send-1.2.1.tgz",
      "integrity": "sha512-1gnZf7DFcoIcajTjTwjwuDjzuz4PPcY2StKPlsGAQ1+YH20IRVrBaXSWmdjowTJ6u8Rc01PoYOGHXfP1mYcZNQ==",
      "license": "MIT",
      "dependencies": {
        "debug": "^4.4.3",
        "encodeurl": "^2.0.0",
        "escape-html": "^1.0.3",
        "etag": "^1.8.1",
        "fresh": "^2.0.0",
        "http-errors": "^2.0.1",
        "mime-types": "^3.0.2",
        "ms": "^2.1.3",
        "on-finished": "^2.4.1",
        "range-parser": "^1.2.1",
        "statuses": "^2.0.2"
      },
      "engines": {
        "node": ">= 18"
      },
      "funding": {
        "type": "opencollective",
        "url": "https://opencollective.com/express"
      }
    },
    "node_modules/serve-static": {
      "version": "2.2.1",
      "resolved": "https://registry.npmjs.org/serve-static/-/serve-static-2.2.1.tgz",
      "integrity": "sha512-xRXBn0pPqQTVQiC8wyQrKs2MOlX24zQ0POGaj0kultvoOCstBQM5yvOhAVSUwOMjQtTvsPWoNCHfPGwaaQJhTw==",
      "license": "MIT",
      "dependencies": {
        "encodeurl": "^2.0.0",
        "escape-html": "^1.0.3",
        "parseurl": "^1.3.3",
        "send": "^1.2.0"
      },
      "engines": {
        "node": ">= 18"
      },
      "funding": {
        "type": "opencollective",
        "url": "https://opencollective.com/express"
      }
    },
    "node_modules/setprototypeof": {
      "version": "1.2.0",
      "resolved": "https://registry.npmjs.org/setprototypeof/-/setprototypeof-1.2.0.tgz",
      "integrity": "sha512-E5LDX7Wrp85Kil5bhZv46j8jOeboKq5JMmYM3gVGdGH8xFpPWXUMsNrlODCrkoxMEeNi/XZIwuRvY4XNwYMJpw==",
      "license": "ISC"
    },
    "node_modules/side-channel": {
      "version": "1.1.1",
      "resolved": "https://registry.npmjs.org/side-channel/-/side-channel-1.1.1.tgz",
      "integrity": "sha512-6x6dK6zJdpTzF4sQeNYxwtvBzf6Eg4GtlesS94HOvTudUeyK2WXAaIfmDgsyslYrRBeFIlsi54AYsFGUuhmvrQ==",
      "license": "MIT",
      "dependencies": {
        "es-errors": "^1.3.0",
        "object-inspect": "^1.13.4",
        "side-channel-list": "^1.0.1",
        "side-channel-map": "^1.0.1",
        "side-channel-weakmap": "^1.0.2"
      },
      "engines": {
        "node": ">= 0.4"
      },
      "funding": {
        "url": "https://github.com/sponsors/ljharb"
      }
    },
    "node_modules/side-channel-list": {
      "version": "1.0.1",
      "resolved": "https://registry.npmjs.org/side-channel-list/-/side-channel-list-1.0.1.tgz",
      "integrity": "sha512-mjn/0bi/oUURjc5Xl7IaWi/OJJJumuoJFQJfDDyO46+hBWsfaVM65TBHq2eoZBhzl9EchxOijpkbRC8SVBQU0w==",
      "license": "MIT",
      "dependencies": {
        "es-errors": "^1.3.0",
        "object-inspect": "^1.13.4"
      },
      "engines": {
        "node": ">= 0.4"
      },
      "funding": {
        "url": "https://github.com/sponsors/ljharb"
      }
    },
    "node_modules/side-channel-map": {
      "version": "1.0.1",
      "resolved": "https://registry.npmjs.org/side-channel-map/-/side-channel-map-1.0.1.tgz",
      "integrity": "sha512-VCjCNfgMsby3tTdo02nbjtM/ewra6jPHmpThenkTYh8pG9ucZ/1P8So4u4FGBek/BjpOVsDCMoLA/iuBKIFXRA==",
      "license": "MIT",
      "dependencies": {
        "call-bound": "^1.0.2",
        "es-errors": "^1.3.0",
        "get-intrinsic": "^1.2.5",
        "object-inspect": "^1.13.3"
      },
      "engines": {
        "node": ">= 0.4"
      },
      "funding": {
        "url": "https://github.com/sponsors/ljharb"
      }
    },
    "node_modules/side-channel-weakmap": {
      "version": "1.0.2",
      "resolved": "https://registry.npmjs.org/side-channel-weakmap/-/side-channel-weakmap-1.0.2.tgz",
      "integrity": "sha512-WPS/HvHQTYnHisLo9McqBHOJk2FkHO/tlpvldyrnem4aeQp4hai3gythswg6p01oSoTl58rcpiFAjF2br2Ak2A==",
      "license": "MIT",
      "dependencies": {
        "call-bound": "^1.0.2",
        "es-errors": "^1.3.0",
        "get-intrinsic": "^1.2.5",
        "object-inspect": "^1.13.3",
        "side-channel-map": "^1.0.1"
      },
      "engines": {
        "node": ">= 0.4"
      },
      "funding": {
        "url": "https://github.com/sponsors/ljharb"
      }
    },
    "node_modules/statuses": {
      "version": "2.0.2",
      "resolved": "https://registry.npmjs.org/statuses/-/statuses-2.0.2.tgz",
      "integrity": "sha512-DvEy55V3DB7uknRo+4iOGT5fP1slR8wQohVdknigZPMpMstaKJQWhwiYBACJE3Ul2pTnATihhBYnRhZQHGBiRw==",
      "license": "MIT",
      "engines": {
        "node": ">= 0.8"
      }
    },
    "node_modules/toidentifier": {
      "version": "1.0.1",
      "resolved": "https://registry.npmjs.org/toidentifier/-/toidentifier-1.0.1.tgz",
      "integrity": "sha512-o5sSPKEkg/DIQNmH43V0/uerLrpzVedkUh8tGNvaeXpfpuwjKenlSox/2O/BTlZUtEe+JG7s5YhEz608PlAHRA==",
      "license": "MIT",
      "engines": {
        "node": ">=0.6"
      }
    },
    "node_modules/type-is": {
      "version": "2.1.0",
      "resolved": "https://registry.npmjs.org/type-is/-/type-is-2.1.0.tgz",
      "integrity": "sha512-faYHw0anBbc/kWF3zFTEnxSFOAGUX9GFbOBthvDdLsIlEoWOFOtS0zgCiQYwIskL9iGXZL3kAXD8OoZ4GmMATA==",
      "license": "MIT",
      "dependencies": {
        "content-type": "^2.0.0",
        "media-typer": "^1.1.0",
        "mime-types": "^3.0.0"
      },
      "engines": {
        "node": ">= 18"
      },
      "funding": {
        "type": "opencollective",
        "url": "https://opencollective.com/express"
      }
    },
    "node_modules/type-is/node_modules/content-type": {
      "version": "2.0.0",
      "resolved": "https://registry.npmjs.org/content-type/-/content-type-2.0.0.tgz",
      "integrity": "sha512-j/O/d7GcZCyNl7/hwZAb606rzqkyvaDctLmckbxLzHvFBzTJHuGEdodATcP3yIRoDrLHkIATJuvzbFlp/ki2cQ==",
      "license": "MIT",
      "engines": {
        "node": ">=18"
      },
      "funding": {
        "type": "opencollective",
        "url": "https://opencollective.com/express"
      }
    },
    "node_modules/undici-types": {
      "version": "8.3.0",
      "resolved": "https://registry.npmjs.org/undici-types/-/undici-types-8.3.0.tgz",
      "integrity": "sha512-j375ScV60dom+YkPFIfTLcOiPxkN/buHz5GobjLhixFuANaNs3C9l4GmrWqejgXWJ7BbJcFYpTEUkS1Ge8bpZQ==",
      "license": "MIT"
    },
    "node_modules/unpipe": {
      "version": "1.0.0",
      "resolved": "https://registry.npmjs.org/unpipe/-/unpipe-1.0.0.tgz",
      "integrity": "sha512-pjy2bYhSsufwWlKwPc+l3cN7+wuJlK6uz0YdJEOlQDbl6jo/YlPi4mb8agUkVC8BF7V8NuzeyPNqRksA3hztKQ==",
      "license": "MIT",
      "engines": {
        "node": ">= 0.8"
      }
    },
    "node_modules/vary": {
      "version": "1.1.2",
      "resolved": "https://registry.npmjs.org/vary/-/vary-1.1.2.tgz",
      "integrity": "sha512-BNGbWLfd0eUPabhkXUVm0j8uuvREyTh5ovRa/dyow/BqAbZJyC+5fU+IzQOzmAKzYqYRAISoRhdQr3eIZ/PXqg==",
      "license": "MIT",
      "engines": {
        "node": ">= 0.8"
      }
    },
    "node_modules/web-streams-polyfill": {
      "version": "3.3.3",
      "resolved": "https://registry.npmjs.org/web-streams-polyfill/-/web-streams-polyfill-3.3.3.tgz",
      "integrity": "sha512-d2JWLCivmZYTSIoge9MsgFCZrt571BikcWGYkjC1khllbTeDlGqZ2D8vD8E/lJa8WGWbb7Plm8/XJYV7IJHZZw==",
      "license": "MIT",
      "engines": {
        "node": ">= 8"
      }
    },
    "node_modules/wrappy": {
      "version": "1.0.2",
      "resolved": "https://registry.npmjs.org/wrappy/-/wrappy-1.0.2.tgz",
      "integrity": "sha512-l4Sp/DRseor9wL6EvV2+TuQn63dMkPjZ/sp9XkghTEbV9KlPS1xUsZ3u7/IQO4wxtcFB4bgpQPRcR3QCvezPcQ==",
      "license": "ISC"
    },
    "node_modules/ws": {
      "version": "8.21.1",
      "resolved": "https://registry.npmjs.org/ws/-/ws-8.21.1.tgz",
      "integrity": "sha512-+0NTnW77fFN/DjQi6k/Sq/Yvk4Sgajw7urW8V+asjXnRgDs9gyGkdb7EzgfhA4goXsRIZKE28fzIXBHEzhuiWw==",
      "license": "MIT",
      "engines": {
        "node": ">=10.0.0"
      },
      "peerDependencies": {
        "bufferutil": "^4.0.1",
        "utf-8-validate": ">=5.0.2"
      },
      "peerDependenciesMeta": {
        "bufferutil": {
          "optional": true
        },
        "utf-8-validate": {
          "optional": true
        }
      }
    }
  }
}
{
  "name": "ai-chat-microservice",
  "version": "1.0.0",
  "description": "An AI-powered educational chat microservice built with Node.js and Gemini AI",
  "main": "server.js",
  "scripts": {
    "start": "node server.js",
    "dev": "node --watch server.js"
  },
  "keywords": [
    "nodejs",
    "express",
    "gemini-ai",
    "api"
  ],
  "author": "",
  "license": "ISC",
  "dependencies": {
    "@google/genai": "^2.13.0",
    "cors": "^2.8.5",
    "express": "^4.19.2"
  }
}
const express = require('express');
const cors = require('cors');

// Import official Google Gen AI SDK
const { GoogleGenAI } = require('@google/genai');

const app = express();
const PORT = 3000;

// Enable CORS and JSON parsing middleware
app.use(cors());
app.use(express.json());

// Hardcoded Gemini API Key
const GEMINI_API_KEY = "AQ.Ab8RN6IfJekCSeBoJ7C2RPOc0l9MktZJUbJrs_dRwYcZR5f18g"; 

let aiClient = null;

if (GEMINI_API_KEY) {
  try {
    aiClient = new GoogleGenAI({ apiKey: GEMINI_API_KEY });
    console.log("✅ Gemini AI Client successfully connected!");
  } catch (err) {
    console.error("❌ Error initializing Gemini Client:", err.message);
  }
} else {
  console.warn("⚠️ Valid GEMINI_API_KEY is missing in server.js!");
}

// AI Chat Endpoint returning structured JSON
app.post('/api/node-chat', async (req, res) => {
  const { message, grade_semester } = req.body;

  if (!message || message.trim() === '') {
    return res.status(400).json({ 
      success: false, 
      error: 'Message field is required in JSON body.' 
    });
  }

  if (!aiClient) {
    return res.status(500).json({
      success: false,
      error: "API key is missing or invalid in server.js.",
      reply: "⚠️ GEMINI_API_KEY is not properly configured."
    });
  }

  try {
    console.log(`📩 Processing Prompt: "${message}"`);

    const systemInstruction = `You are a helpful, highly intelligent AI Assistant. 
Respond accurately, dynamically, and naturally to the user's prompt. 
Context/Course info if applicable: "${grade_semester || 'General Education'}".`;

    // Send request using Gemini 3.5 Flash
    const response = await aiClient.models.generateContent({
      model: 'gemini-3.5-flash',
      contents: message,
      config: {
        systemInstruction: systemInstruction,
      }
    });

    const aiReply = response.text ? response.text.trim() : '';
    console.log("✨ Successfully received live AI response.");

    return res.json({
      success: true,
      timestamp: new Date().toISOString(),
      prompt: message,
      reply: aiReply
    });

  } catch (error) {
    console.error('❌ Gemini Execution Error:', error.message);
    return res.status(500).json({
      success: false,
      error: error.message,
      reply: `⚠️ AI Processing Error: ${error.message}`
    });
  }
});

app.listen(PORT, () => {
  console.log(`=======================================================`);
  console.log(`🤖 Real AI Chat Microservice running at: http://127.0.0.1:${PORT}`);
  console.log(`=======================================================`);
});
