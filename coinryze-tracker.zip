import os
import zipfile

project_name = "coinryze-tracker"

structure = {
    "backend": {
        "app.py": """from fastapi import FastAPI
from fastapi.middleware.cors import CORSMiddleware
from backend.fetcher import fetch_coinryze
from backend.utils import helpers

app = FastAPI(title="CoinRyze Tracker Backend")

app.add_middleware(
    CORSMiddleware,
    allow_origins=["*"],
    allow_methods=["*"],
    allow_headers=["*"],
)

@app.get("/")
async def home():
    return {"message": "CoinRyze Tracker Backend Running"}

@app.get("/latest")
async def latest_draw():
    data = fetch_coinryze.get_latest_draw()
    return {"latest_draw": data}

@app.get("/accuracy")
async def accuracy():
    return {"accuracy": helpers.calculate_accuracy()}
""",
        "fetcher": {
            "fetch_coinryze.py": """import csv
import os
import random
from backend.utils import helpers

DATA_FILE = os.path.join(os.path.dirname(__file__), "..", "data", "seed.csv")

def get_latest_draw():
    with open(DATA_FILE, "r") as f:
        lines = f.readlines()
        last_line = lines[-1].strip().split(",")
    new_draw_number = str(int(last_line[0]) + 1)
    new_result = str(random.randint(0, 9))
    new_line = new_draw_number + "," + new_result + "\\n"
    with open(DATA_FILE, "a") as f:
        f.write(new_line)
    helpers.sync_to_r2(new_line)
    return [new_draw_number, new_result]
"""
        },
        "utils": {
            "helpers.py": """import os
import requests

R2_BUCKET = 'YOUR_R2_BUCKET'
R2_ENDPOINT = 'https://YOUR_ACCOUNT_ID.r2.cloudflarestorage.com'
R2_KEY_ID = 'YOUR_R2_KEY_ID'
R2_SECRET = 'YOUR_R2_SECRET'

def sync_to_r2(line):
    filename = 'seed.csv'
    file_path = os.path.join(os.path.dirname(__file__), "..", "data", filename)
    try:
        with open(file_path, 'rb') as f:
            requests.put(
                f"{R2_ENDPOINT}/{filename}",
                data=f,
                auth=(R2_KEY_ID, R2_SECRET)
            )
    except Exception as e:
        print("R2 sync failed:", e)

def calculate_accuracy():
    # Dummy accuracy calculation for demo
    return round(random.random() * 100, 2)
"""
        },
        "data": {
            "seed.csv": "draw_number,result\n1,5\n2,3\n3,7\n"
        }
    },
    "tg_listener": {
        "tg_listener.py": """from telethon import TelegramClient, events
from backend.fetcher import fetch_coinryze

api_id = 'YOUR_API_ID'
api_hash = 'YOUR_API_HASH'
session_name = 'my_session'

client = TelegramClient(session_name, api_id, api_hash)

@client.on(events.NewMessage)
async def handler(event):
    print(f"New message from {event.chat_id}: {event.text}")
    # Trigger fetch on certain messages
    if 'draw' in event.text.lower():
        new_draw = fetch_coinryze.get_latest_draw()
        print("New draw added:", new_draw)

client.start()
print("Telegram listener started...")
client.run_until_disconnected()
"""
    },
    "frontend": {
        "dashboard.py": """import streamlit as st
import pandas as pd
import os
import requests
import time

DATA_FILE = os.path.join("..","backend","data","seed.csv")
API_URL = "http://localhost:8000/latest"

st.title("CoinRyze Tracker Dashboard")

@st.cache_data(ttl=10)
def load_data():
    df = pd.read_csv(DATA_FILE)
    return df

df = load_data()
st.dataframe(df)

st.line_chart(df['result'])

st.subheader("Prediction Accuracy")
accuracy_url = "http://localhost:8000/accuracy"
acc = requests.get(accuracy_url).json().get('accuracy', 0)
st.metric("Accuracy %", acc)
"""
    },
    "requirements.txt": """fastapi
uvicorn
pandas
streamlit
telethon
requests
python-multipart
"""
}

# --- Function to create folders and files ---
def create_structure(base_path, struct):
    for name, content in struct.items():
        path = os.path.join(base_path, name)
        if isinstance(content, dict):
            os.makedirs(path, exist_ok=True)
            create_structure(path, content)
        else:
            os.makedirs(os.path.dirname(path), exist_ok=True)
            with open(path, "w", encoding="utf-8") as f:
                f.write(content)

# --- Create project ---
create_structure(".", structure)

# --- Zip project ---
zip_filename = project_name + ".zip"
with zipfile.ZipFile(zip_filename, 'w', zipfile.ZIP_DEFLATED) as zipf:
    for root, dirs, files in os.walk(project_name):
        for file in files:
            file_path = os.path.join(root, file)
            zipf.write(file_path, os.path.relpath(file_path, project_name))

print(f"{zip_filename} created successfully!")
