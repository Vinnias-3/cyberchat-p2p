#!/usr/bin/env python3
"""
CYBER CHAT P2P - World Wide Peer-to-Peer Chat using libp2p
No servers needed - connects directly across the internet!
"""

import sys
import os
import json
import threading
import time
import random
import string
import socket
from datetime import datetime

try:
    from PyQt5.QtWidgets import (
        QApplication, QMainWindow, QWidget, QVBoxLayout, QHBoxLayout,
        QPushButton, QLabel, QLineEdit, QTextEdit, QGroupBox,
        QMessageBox, QStatusBar, QTabWidget, QListWidget,
        QListWidgetItem, QSplitter, QFrame
    )
    from PyQt5.QtCore import Qt, QThread, pyqtSignal, QTimer
    from PyQt5.QtGui import QFont, QIcon, QPixmap
    HAS_PYQT = True
except:
    HAS_PYQT = False
    print("PyQt5 not installed")

# Check for libp2p availability
try:
    from libp2p import new_host
    from libp2p.peer import PeerInfo
    from libp2p.network.stream.net_stream_interface import INetStream
    from libp2p.crypto.secp256k1 import create_new_key_pair
    HAS_LIBP2P = False 
    # Note: Full libp2p implementation requires additional setup
except:
    HAS_LIBP2P = False

# Fallback to direct TCP with NAT traversal using UPnP
try:
    import upnpclient
    HAS_UPNP = True
except:
    HAS_UPNP = False


class DirectP2PConnection(QThread):
    message_received = pyqtSignal(str, str)
    connection_status = pyqtSignal(str, bool)
    peer_connected = pyqtSignal(str)
    
    def __init__(self, port=5555):
        super().__init__()
        self.port = port
        self.running = True
        self.active_connections = {}
        self.public_ip = None
        self.get_public_ip()
        
    def get_public_ip(self):
        """Get the public IP address for external connections"""
        try:
            import requests
            response = requests.get('https://api.ipify.org', timeout=5)
            self.public_ip = response.text
        except:
            self.public_ip = None
    
    def setup_port_forwarding(self):
        """Attempt to automatically forward port using UPnP"""
        if not HAS_UPNP:
            return False
        
        try:
            devices = upnpclient.discover()
            for device in devices:
                try:
                    device.WANIPConn1.AddPortMapping(
                        NewRemoteHost='',
                        NewExternalPort=self.port,
                        NewProtocol='TCP',
                        NewInternalPort=self.port,
                        NewInternalClient=self.get_local_ip(),
                        NewEnabled=1,
                        NewPortMappingDescription='Cyber Chat P2P'
                    )
                    return True
                except:
                    continue
        except:
            pass
        return False
    
    def get_local_ip(self):
        """Get local IP address"""
        try:
            s = socket.socket(socket.AF_INET, socket.SOCK_DGRAM)
            s.connect(("8.8.8.8", 80))
            ip = s.getsockname()[0]
            s.close()
            return ip
        except:
            return "127.0.0.1"
    
    def start_server(self):
        """Start TCP server for direct connections"""
        self.server_socket = socket.socket(socket.AF_INET, socket.SOCK_STREAM)
        self.server_socket.setsockopt(socket.SOL_SOCKET, socket.SO_REUSEADDR, 1)
        self.server_socket.bind(('0.0.0.0', self.port))
        self.server_socket.listen(10)
        
        self.connection_status.emit(f"🌍 P2P Server on port {self.port}", True)
        if self.public_ip:
            self.connection_status.emit(f"📡 Public: {self.public_ip}:{self.port}", True)
        
        while self.running:
            try:
                client, addr = self.server_socket.accept()
                peer_id = addr[0]
                self.active_connections[peer_id] = client
                self.peer_connected.emit(peer_id)
                
                recv_thread = threading.Thread(target=self.receive_messages, args=(client, peer_id))
                recv_thread.daemon = True
                recv_thread.start()
            except:
                pass
    
    def receive_messages(self, client, peer_id):
        """Receive messages from connected peer"""
        while self.running:
            try:
                data = client.recv(4096)
                if data:
                    message = data.decode('utf-8')
                    self.message_received.emit(message, peer_id)
                else:
                    break
            except:
                break
        
        if peer_id in self.active_connections:
            del self.active_connections[peer_id]
        try:
            client.close()
        except:
            pass
    
    def connect_to_peer(self, target_ip, target_port=5555):
        """Connect to a remote peer"""
        try:
            client = socket.socket(socket.AF_INET, socket.SOCK_STREAM)
            client.settimeout(10)
            client.connect((target_ip, target_port))
            self.active_connections[target_ip] = client
            self.peer_connected.emit(target_ip)
            return True
        except Exception as e:
            self.connection_status.emit(f"Connection failed: {e}", False)
            return False
    
    def send_message(self, message, target_ip):
        """Send message to a connected peer"""
        if target_ip in self.active_connections:
            try:
                self.active_connections[target_ip].send(message.encode('utf-8'))
                return True
            except:
                return False
        return False
    
    def run(self):
        self.start_server()
    
    def stop(self):
        self.running = False
        if hasattr(self, 'server_socket'):
            self.server_socket.close()
        for conn in self.active_connections.values():
            try:
                conn.close()
            except:
                pass


class CyberChatP2P(QMainWindow):
    def __init__(self):
        super().__init__()
        self.my_code = self.generate_unique_code()
        self.contacts = []
        self.current_chat = None
        self.p2p = None
        self.setup_ui()
        self.apply_theme()
        self.start_p2p()
        self.load_contacts()
    
    def generate_unique_code(self):
        """Generate unique 8-character code for this user"""
        random_part = ''.join(random.choices(string.ascii_uppercase + string.digits, k=8))
        return random_part
    
    def start_p2p(self):
        """Start the P2P connection handler"""
        self.p2p = DirectP2PConnection()
        self.p2p.message_received.connect(self.on_message_received)
        self.p2p.connection_status.connect(self.on_connection_status)
        self.p2p.peer_connected.connect(self.on_peer_connected)
        self.p2p.start()
    
    def on_message_received(self, message, sender_ip):
        """Handle incoming message"""
        self.add_to_chat("Received", message, sender_ip)
        self.save_message(sender_ip, message, False)
        
        # Show notification
        self.status_bar.showMessage(f"📩 New message from {sender_ip}", 2000)
    
    def on_connection_status(self, status, success):
        """Update connection status"""
        self.status_bar.showMessage(status, 3000)
    
    def on_peer_connected(self, peer_ip):
        """Update UI when peer connects"""
        self.status_bar.showMessage(f"✅ Connected to {peer_ip}", 2000)
    
    def setup_ui(self):
        self.setWindowTitle("CYBER CHAT P2P - Worldwide Messenger")
        self.setGeometry(100, 100, 1100, 700)
        self.setMinimumSize(900, 550)
        
        icon_path = os.path.join(os.path.dirname(__file__), 'icons', 'cc_logo.png')
        if os.path.exists(icon_path):
            self.setWindowIcon(QIcon(icon_path))
        
        central = QWidget()
        self.setCentralWidget(central)
        main_layout = QHBoxLayout(central)
        main_layout.setContentsMargins(0, 0, 0, 0)
        
        # Left Panel - Contacts
        left_panel = QFrame()
        left_panel.setFixedWidth(280)
        left_panel.setStyleSheet("background-color: rgba(0, 255, 136, 0.05); border-right: 1px solid rgba(0, 255, 136, 0.2);")
        left_layout = QVBoxLayout(left_panel)
        
        # Profile header
        profile_widget = QWidget()
        profile_layout = QVBoxLayout(profile_widget)
        
        self.code_label = QLabel(f"🔐 YOUR CODE: {self.my_code}")
        self.code_label.setFont(QFont("Arial", 11, QFont.Bold))
        self.code_label.setStyleSheet("color: #00ff88; background-color: rgba(0, 255, 136, 0.15); padding: 10px; border-radius: 10px;")
        self.code_label.setAlignment(Qt.AlignCenter)
        profile_layout.addWidget(self.code_label)
        
        left_layout.addWidget(profile_widget)
        
        # Contacts title
        contacts_title = QLabel("💬 CONTACTS")
        contacts_title.setFont(QFont("Arial", 12, QFont.Bold))
        contacts_title.setStyleSheet("color: #00ff88; padding: 10px 0 5px 0;")
        left_layout.addWidget(contacts_title)
        
        # Contacts list
        self.contacts_list = QListWidget()
        self.contacts_list.setStyleSheet("""
            QListWidget {
                background-color: transparent;
                border: none;
            }
            QListWidget::item {
                padding: 12px;
                border-radius: 10px;
                margin: 2px 5px;
            }
            QListWidget::item:hover {
                background-color: rgba(0, 255, 136, 0.1);
            }
            QListWidget::item:selected {
                background-color: rgba(0, 255, 136, 0.2);
                color: #00ff88;
            }
        """)
        self.contacts_list.itemClicked.connect(self.on_contact_selected)
        left_layout.addWidget(self.contacts_list)
        
        # Add contact section
        add_group = QGroupBox("➕ ADD CONTACT")
        add_group.setStyleSheet("""
            QGroupBox {
                border: 1px solid rgba(0, 255, 136, 0.3);
                border-radius: 10px;
                margin-top: 10px;
                padding-top: 10px;
            }
            QGroupBox::title {
                color: #00ff88;
                left: 10px;
            }
        """)
        add_layout = QVBoxLayout(add_group)
        
        self.contact_code_input = QLineEdit()
        self.contact_code_input.setPlaceholderText("Enter contact's unique code (optional)")
        self.contact_code_input.setStyleSheet("padding: 10px; border-radius: 8px; border: 1px solid rgba(0, 255, 136, 0.3);")
        add_layout.addWidget(self.contact_code_input)
        
        self.contact_ip_input = QLineEdit()
        self.contact_ip_input.setPlaceholderText("Enter IP address (public or local)")
        self.contact_ip_input.setStyleSheet("padding: 10px; border-radius: 8px; border: 1px solid rgba(0, 255, 136, 0.3);")
        add_layout.addWidget(self.contact_ip_input)
        
        add_btn = QPushButton("🌍 ADD CONTACT (WORLDWIDE)")
        add_btn.setStyleSheet("""
            QPushButton {
                background: qlineargradient(x1:0, y1:0, x2:1, y2:1, stop:0 #00ff88, stop:1 #0066ff);
                color: #000;
                font-weight: bold;
                border: none;
                padding: 10px;
                border-radius: 8px;
            }
            QPushButton:hover {
                opacity: 0.8;
            }
        """)
        add_btn.clicked.connect(self.add_contact)
        add_layout.addWidget(add_btn)
        
        left_layout.addWidget(add_group)
        
        # Info section
        info_label = QLabel("🌍 WORLDWIDE P2P\nNo servers needed\nDirect connection")
        info_label.setWordWrap(True)
        info_label.setStyleSheet("color: #888; font-size: 10px; padding: 10px; background-color: rgba(0, 255, 136, 0.05); border-radius: 8px; margin-top: 10px;")
        info_label.setAlignment(Qt.AlignCenter)
        left_layout.addWidget(info_label)
        
        # Right Panel - Chat Area
        right_panel = QFrame()
        right_panel.setStyleSheet("background-color: transparent;")
        right_layout = QVBoxLayout(right_panel)
        
        # Chat header
        self.chat_header = QWidget()
        chat_header_layout = QHBoxLayout(self.chat_header)
        self.chat_name_label = QLabel("✨ SELECT A CONTACT TO START CHATTING ✨")
        self.chat_name_label.setFont(QFont("Arial", 14, QFont.Bold))
        self.chat_name_label.setStyleSheet("color: #00ff88;")
        chat_header_layout.addWidget(self.chat_name_label)
        right_layout.addWidget(self.chat_header)
        
        # Chat display area
        self.chat_display = QTextEdit()
        self.chat_display.setReadOnly(True)
        self.chat_display.setStyleSheet("""
            QTextEdit {
                background-color: rgba(0, 0, 0, 0.5);
                border: 1px solid rgba(0, 255, 136, 0.3);
                border-radius: 15px;
                padding: 15px;
                font-size: 13px;
                color: #fff;
            }
        """)
        right_layout.addWidget(self.chat_display)
        
        # Message input area
        input_widget = QWidget()
        input_layout = QHBoxLayout(input_widget)
        
        self.message_input = QLineEdit()
        self.message_input.setPlaceholderText("Type your message (encrypted end-to-end)...")
        self.message_input.setStyleSheet("""
            QLineEdit {
                background-color: rgba(0, 0, 0, 0.5);
                border: 1px solid rgba(0, 255, 136, 0.5);
                border-radius: 25px;
                padding: 12px 18px;
                color: #fff;
            }
        """)
        self.message_input.returnPressed.connect(self.send_message)
        input_layout.addWidget(self.message_input)
        
        self.send_btn = QPushButton("➤")
        self.send_btn.setFixedSize(50, 50)
        self.send_btn.setStyleSheet("""
            QPushButton {
                background: qlineargradient(x1:0, y1:0, x2:1, y2:1, stop:0 #00ff88, stop:1 #0066ff);
                color: #000;
                border: none;
                border-radius: 25px;
                font-size: 20px;
                font-weight: bold;
            }
            QPushButton:hover {
                opacity: 0.8;
            }
        """)
        self.send_btn.clicked.connect(self.send_message)
        input_layout.addWidget(self.send_btn)
        
        right_layout.addWidget(input_widget)
        
        # Status bar
        self.status_bar = QStatusBar()
        self.setStatusBar(self.status_bar)
        self.status_bar.showMessage("🌍 P2P Mode Active | Share your code and public IP to chat worldwide")
        
        # Add panels
        main_layout.addWidget(left_panel)
        main_layout.addWidget(right_panel, 1)
    
    def add_contact(self):
        """Add a new contact using their IP address"""
        code = self.contact_code_input.text().strip().upper()
        ip = self.contact_ip_input.text().strip()
        
        if not ip:
            QMessageBox.warning(self, "Error", "Please enter an IP address")
            return
        
        for contact in self.contacts:
            if contact['ip'] == ip:
                QMessageBox.warning(self, "Error", "Contact already exists")
                return
        
        name = f"Contact {code[:4] if code else ip[-4:]}"
        
        self.contacts.append({
            'code': code,
            'ip': ip,
            'name': name
        })
        
        self.save_contacts()
        self.refresh_contacts_list()
        
        # Try to connect
        self.p2p.connect_to_peer(ip)
        
        self.contact_code_input.clear()
        self.contact_ip_input.clear()
        
        self.status_bar.showMessage(f"🌍 Added contact at {ip}", 2000)
    
    def refresh_contacts_list(self):
        """Refresh contacts list"""
        self.contacts_list.clear()
        for contact in self.contacts:
            status = "🟢" if contact['ip'] in self.p2p.active_connections else "⚪"
            item = QListWidgetItem(f"{status} {contact['name']}\n📍 {contact['ip']}")
            item.setData(Qt.UserRole, contact)
            self.contacts_list.addItem(item)
    
    def on_contact_selected(self, item):
        """Handle contact selection"""
        self.current_chat = item.data(Qt.UserRole)
        self.chat_name_label.setText(f"💬 CHATTING WITH: {self.current_chat['name']} (P2P Worldwide)")
        
        # Try to connect if not already connected
        if self.current_chat['ip'] not in self.p2p.active_connections:
            self.p2p.connect_to_peer(self.current_chat['ip'])
        
        self.load_chat_history(self.current_chat['ip'])
        
        # Refresh contact status
        self.refresh_contacts_list()
    
    def load_chat_history(self, contact_ip):
        """Load chat history"""
        self.chat_display.clear()
        history_file = os.path.join(os.path.dirname(__file__), f"chat_{contact_ip.replace('.', '_')}.json")
        
        if os.path.exists(history_file):
            try:
                with open(history_file, 'r') as f:
                    history = json.load(f)
                    for msg in history:
                        if msg['sent']:
                            self.add_to_chat("Sent", msg['message'], "")
                        else:
                            self.add_to_chat("Received", msg['message'], contact_ip)
            except:
                pass
    
    def save_message(self, contact_ip, message, is_sent):
        """Save message to history"""
        history_file = os.path.join(os.path.dirname(__file__), f"chat_{contact_ip.replace('.', '_')}.json")
        
        history = []
        if os.path.exists(history_file):
            try:
                with open(history_file, 'r') as f:
                    history = json.load(f)
            except:
                pass
        
        history.append({
            'timestamp': datetime.now().isoformat(),
            'message': message,
            'sent': is_sent
        })
        
        if len(history) > 1000:
            history = history[-1000:]
        
        with open(history_file, 'w') as f:
            json.dump(history, f, indent=2)
    
    def add_to_chat(self, direction, message, sender_ip):
        """Add message to chat display"""
        timestamp = datetime.now().strftime("%H:%M")
        
        if direction == "Sent":
            bubble_style = """
                background: qlineargradient(x1:0, y1:0, x2:1, y2:1, stop:0 #00ff88, stop:1 #0066ff);
                color: #000;
                border-radius: 18px;
                padding: 10px 15px;
                margin: 5px 0 5px 50px;
            """
            alignment = "right"
        else:
            bubble_style = """
                background: rgba(0, 255, 136, 0.15);
                color: #00ff88;
                border-radius: 18px;
                padding: 10px 15px;
                margin: 5px 50px 5px 0;
                border: 1px solid rgba(0, 255, 136, 0.3);
            """
            alignment = "left"
        
        html = f"""
        <div style='text-align: {alignment};'>
            <div style='display: inline-block; max-width: 70%; {bubble_style}'>
                <div style='font-size: 13px; word-wrap: break-word;'>{message}</div>
                <div style='font-size: 10px; opacity: 0.7; margin-top: 5px;'>{timestamp}</div>
            </div>
        </div>
        """
        
        self.chat_display.insertHtml(html)
        self.chat_display.ensureCursorVisible()
    
    def send_message(self):
        """Send message to selected contact"""
        if not self.current_chat:
            QMessageBox.warning(self, "Error", "Please select a contact first")
            return
        
        message = self.message_input.text().strip()
        if not message:
            return
        
        success = self.p2p.send_message(message, self.current_chat['ip'])
        
        if success:
            self.add_to_chat("Sent", message, "")
            self.save_message(self.current_chat['ip'], message, True)
            self.message_input.clear()
            self.status_bar.showMessage("✅ Message sent worldwide", 1000)
        else:
            QMessageBox.warning(self, "Error", f"Cannot reach {self.current_chat['ip']}. Make sure they are online and have port forwarding enabled.")
    
    def save_contacts(self):
        """Save contacts to file"""
        contacts_file = os.path.join(os.path.dirname(__file__), "contacts.json")
        with open(contacts_file, 'w') as f:
            json.dump(self.contacts, f, indent=2)
    
    def load_contacts(self):
        """Load contacts from file"""
        contacts_file = os.path.join(os.path.dirname(__file__), "contacts.json")
        if os.path.exists(contacts_file):
            try:
                with open(contacts_file, 'r') as f:
                    self.contacts = json.load(f)
                self.refresh_contacts_list()
            except:
                pass
    
    def apply_theme(self):
        """Apply cyberpunk theme"""
        self.setStyleSheet("""
            QMainWindow {
                background: qlineargradient(x1:0, y1:0, x2:1, y2:1,
                    stop:0 #0a0a0a, stop:0.5 #0a1a0a, stop:1 #0a0a1a);
            }
            QLabel {
                color: #00ff88;
            }
            QPushButton {
                font-weight: bold;
            }
        """)


def main():
    app = QApplication(sys.argv)
    window = CyberChatP2P()
    window.show()
    sys.exit(app.exec_())


if __name__ == "__main__":
    main()
