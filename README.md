# 🎥 simple_video - Easy Video Creation Tool

[![Download simple_video](https://img.shields.io/badge/Download-Release-green?style=for-the-badge)](https://github.com/Joy-S-07/simple_video/releases)

---

## 📦 What is simple_video?

simple_video is a standalone app designed to help you create videos easily. It uses ComfyUI as a backend. The app guides you through making videos step-by-step, starting from images and adding music and effects. You don't need technical skills to use it.

The app supports multiple tasks in one place:

- Turn images into videos  
- Add music to videos  
- Blend video and music together  
- Create prompts to guide video style and content  

This app focuses on simplicity. It gives you clear steps to create videos without fuss.

---

## ⚙️ System Requirements

Before you start, make sure your Windows PC meets these requirements:

- Windows 10 or later  
- Python 3.10 or newer installed  
- ffmpeg installed and accessible from the system PATH  
- About 4 GB free disk space  
- Internet connection to download packages  

---

## 🚀 Getting Started with simple_video

Follow these steps to download and run simple_video on your Windows computer.

### 1. Download simple_video

Click the button below to visit the release page where you can get the latest version.

[![Download simple_video](https://img.shields.io/badge/Download-Release-blue?style=for-the-badge)](https://github.com/Joy-S-07/simple_video/releases)

Open the page and find the latest release files. Download the version with the `.zip` or `.exe` extension if available.

### 2. Install Python and ffmpeg

If you don’t have Python 3.10 or later, go to https://www.python.org/downloads/ and install it.

Make sure to check **Add Python to PATH** during installation.

Download ffmpeg from https://ffmpeg.org/download.html and follow the instructions to add it to the PATH environment variable. This step lets simple_video find and use ffmpeg from any command window.

### 3. Start ComfyUI

simple_video uses ComfyUI as a backend. You need to run ComfyUI on your computer first.

- Open a command prompt or PowerShell.  
- Start ComfyUI by running its main program. By default, it listens on `127.0.0.1:8188`.  

If you are unsure how to start ComfyUI, check its documentation or installation guide.

### 4. Setup and run simple_video

a. Open a command prompt or PowerShell window.

b. Change directory to where you saved simple_video. For example:

```
cd C:\path\to\simple_video_app
```

c. Install required Python packages:

```
pip install -r requirements.txt
```

d. Start the app by running:

```
start.bat
```

You can run `start.bat` by double-clicking it in File Explorer or by typing the line above.

The app will connect to ComfyUI running on your machine and start its interface.

---

## 📝 How to Use simple_video

After starting simple_video, follow these simple steps inside the app interface:

1. **Create scenario**  
   Write a short description of what you want your video to show. This acts as the main idea or story.

2. **Generate prompts**  
   The app helps you turn your story into detailed prompts for images, video, and music. This makes your video style and content clear.

3. **Create images and video**  
   Use prompts to generate images. Then, combine images to make a video sequence using built-in tools.

4. **Add music**  
   The app can generate music or let you add your own tracks.

5. **Merge video and music**  
   Combine the video and music into a final file.

The app keeps the style consistent using templates and guardrails, so your video looks smooth and stable.

---

## 📂 File and Folder Overview

- `start.bat` — launches the app on Windows  
- `requirements.txt` — list of Python packages to install  
- `.env` file (optional) — add environment variables to customize settings  
- `LICENSE` — MIT license information  
- `README_EN.md` — English version of this guide  

---

## 🔧 Customization and Settings

You can change the app’s behavior by editing a `.env` file in the app folder.

For example, you can set:

- ComfyUI server address if it is different from default `127.0.0.1:8188`  
- Output folder to save generated videos  
- Quality or style presets  

Use a text editor like Notepad to create or edit the `.env` file.

---

## 💡 Tips for Best Results

- Start with a simple scenario to learn how prompts influence videos.  
- Use consistent style templates to keep the video flow stable.  
- Keep ComfyUI running before starting simple_video for better performance.  
- Check that ffmpeg works by typing `ffmpeg -version` in a command prompt.  
- Keep your Python and packages updated regularly.  

---

## 🔗 Resources and Links

- ComfyUI project: https://github.com/comfyanonymous/ComfyUI  
- Python downloads: https://www.python.org/downloads/  
- ffmpeg downloads: https://ffmpeg.org/download.html  
- simple_video releases: https://github.com/Joy-S-07/simple_video/releases  

[![Download simple_video](https://img.shields.io/badge/Download-Release-green?style=for-the-badge)](https://github.com/Joy-S-07/simple_video/releases)