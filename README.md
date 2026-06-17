# AI-Enhanced Group Chat

A full-stack AI-powered group chat application with a FastAPI backend, real-time WebSocket messaging, a self-hosted Llama 3 LLM bot, and an Android wrapper built with Trusted Web Activity (TWA).


---

## Repository Structure

```
ai-enhanced-groupchat/
├── groupchat_app_src/       # FastAPI backend + HTML/CSS/JS frontend
├── embedding/               # Embedding utilities / vector search support
├── llama.cpp/               # Self-hosted LLM inference server (llama.cpp)
```

---

## Features

- **Real-time group chat** via WebSockets
- **AI bot** — send any message ending with `?` to trigger a Llama 3 response
- **JWT authentication** — sign up, log in, stay authenticated
- **MySQL persistence** — all messages and users stored in a MySQL database
- **Android app** — TWA wrapper that loads the web app as a native Android experience

---

## Prerequisites

| Requirement | Version |
|---|---|
| OS | Linux, macOS, or Windows |
| Python | 3.10+ |
| MySQL | 8+ |
| Node.js + npm | Latest LTS (for Bubblewrap CLI, optional) |
| Android Studio | Flamingo or newer |
| JDK | 17 |

---

## Part 1: Web Application

### 1. MySQL Setup

Start MySQL and run the following to create the database and user:

```sql
CREATE DATABASE groupchat CHARACTER SET utf8mb4 COLLATE utf8mb4_unicode_ci;
CREATE USER 'chatuser'@'localhost' IDENTIFIED BY 'chatpass';
GRANT ALL PRIVILEGES ON groupchat.* TO 'chatuser'@'localhost';
FLUSH PRIVILEGES;
```

Or run the provided schema file:

```bash
mysql -u root -p < sql/schema.sql
```

### 2. Backend Setup (FastAPI)

```bash
cd groupchat_app_src/backend
python -m venv .venv
source .venv/bin/activate        # Windows: .venv\Scripts\activate
pip install -r requirements.txt
```

Copy and configure the environment file:

```bash
cp .env.example ../.env
```

Edit `.env` with the following values:

```env
DATABASE_URL=mysql+asyncmy://chatuser:chatpass@localhost:3306/groupchat
JWT_SECRET=<replace with a long random string>
JWT_EXPIRE_MINUTES=43200
LLM_API_BASE=http://localhost:8001/v1
LLM_MODEL=llama-3-8b-instruct
LLM_API_KEY=
APP_HOST=0.0.0.0
APP_PORT=8000
```

Start the server:

```bash
uvicorn app:app --host 0.0.0.0 --port 8000
```

Open [http://localhost:8000](http://localhost:8000), sign up, and log in. Any message containing `?` will trigger the LLM bot.

### 3. LLM Backend — Self-Hosted Llama 3

#### Option A: llama.cpp

```bash
# Already included as a submodule in llama.cpp/
cd llama.cpp
cmake -B build
cmake --build build --config Release
```

Download a Llama 3 GGUF model (e.g. from [bartowski/Meta-Llama-3.1-70B-Instruct-GGUF](https://huggingface.co/bartowski/Meta-Llama-3.1-70B-Instruct-GGUF)) and start the server:

```bash
cd llama.cpp/build/bin
./llama-server -m /path/to/model.gguf --host 0.0.0.0 --port 8001 --ctx-size 4096
```

To enforce an API key:

```bash
./llama-server -m /path/to/model.gguf --host 0.0.0.0 --port 8001 --api-key MYKEY
# Then set LLM_API_KEY=MYKEY in .env
```

#### Option B: vLLM (GPU recommended)

```bash
python -m vllm.entrypoints.openai.api_server \
  --model <your-llama3-model> \
  --host 0.0.0.0 \
  --port 8001
```

Update `.env` with `LLM_API_BASE=http://localhost:8001/v1` for either option, then restart the FastAPI app.

### 4. Frontend

The frontend is plain HTML/CSS/JS served directly by FastAPI as static files — no separate build step required.

---

## Part 2: Android App (TWA)

The Android app wraps the web application using a [Trusted Web Activity](https://developer.chrome.com/docs/android/trusted-web-activity/) so it runs without a browser URL bar.

### Setup

1. Open `twa_android_src/` in Android Studio (Flamingo or newer).
2. Update `app/build.gradle` and `AndroidManifest.xml` with your `applicationId`.
3. Set your deployed web URL in `res/values/strings.xml`:
   ```xml
   <string name="defaultUrl">https://YOUR_DOMAIN/</string>
   ```
4. Build and run on a device or emulator.

### Asset Links (required for full TWA experience)

Generate your signing key fingerprint:

```bash
keytool -list -v -keystore your_keystore.jks -alias your_alias \
  -storepass <storepass> -keypass <keypass>
```

Copy the SHA-256 fingerprint and paste it into `frontend/.well-known/assetlinks.json`. Make sure the `package_name` matches your Android app's `applicationId`. Host this file at `/.well-known/assetlinks.json` on your web server, then reinstall the signed APK.

### Emulators (if no physical device)

- [LDPlayer](https://www.ldplayer.net)
- [BlueStacks](https://www.bluestacks.com/bluestacks-5.html)
- [KoPlayer](https://koplayerpc.com/)

---
