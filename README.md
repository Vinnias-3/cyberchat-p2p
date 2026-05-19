# CyberChat P2P 🌍

Encrypted peer-to-peer messaging application with no servers needed. Connects directly across the internet using libp2p-style networking. Built with PyQt5 GUI and includes a ready-to-install `.deb` package. Developed by **Vinnius Mbuthia**.

## Features
- 🔐 **Encrypted P2P messaging** — no central server
- 🌍 **Worldwide connectivity** — chat across the internet
- 🖥️ **PyQt5 GUI** — professional dark-themed interface
- 📦 **Debian installer** — one-click install on any Debian-based system
- 👥 **Contact management** — save and manage chat contacts
- 📁 **ZIP distribution** — ready-to-share package

## Requirements
- Python 3.6+
- PyQt5
- Linux-based OS (Parrot OS, Kali, Ubuntu, Debian)

## Quick Install

### Option 1: Install via .deb Package
```bash
git clone https://github.com/Vinnias-3/cyberchat-p2p.git
cd cyberchat-p2p/cyber_chat_p2p
sudo dpkg -i cyberchat_installer.deb
Option 2: Run from Source
bash
git clone https://github.com/Vinnias-3/cyberchat-p2p.git
cd cyberchat-p2p/cyber_chat_p2p
pip install PyQt5
python3 main.py
How It Works
Launch CyberChat P2P

Share your peer ID with contacts

Add their peer ID to your contacts list

Start chatting — messages go directly between peers, no server in between

Project Structure
text
cyberchat-p2p/
├── cyber_chat_p2p/              # Main version
│   ├── main.py                  # PyQt5 chat application
│   ├── contacts.json            # Contact storage
│   ├── cyberchat_installer.deb  # Debian installer package
│   ├── CyberChat_Worldwide.zip  # Distribution package
│   └── icons/                   # Application icons
├── cyber_chat_p2p_test2/        # Updated test version
│   ├── main.py
│   ├── contacts.json
│   ├── cyberchat_installer.deb
│   ├── CyberChat_Worldwide.zip
│   └── icons/
└── CyberChat_P2P.zip            # Complete package
Tech Stack
Python 3 — Core language

PyQt5 — GUI framework

Sockets/Networking — Direct P2P communication

JSON — Contact storage

Debian Packaging — .deb installer generation

Author
Vinnius Mbuthia

GitHub: Vinnias-3

Portfolio: vinnius-portfolio-4bcd.vercel.app

Email: techglobal824@gmail.com

License
MIT — Free to use, modify, and distribute.
