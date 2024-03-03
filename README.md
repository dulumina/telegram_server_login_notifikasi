berikut langkah-langkah lengkap beserta kode untuk membuat dan menjalankan bot Telegram yang melaporkan setiap kali ada pengguna yang login di server Linux, termasuk membuatnya otomatis berjalan pada sistem startup menggunakan systemd.

## 1. Buat Bot Telegram
Buka Telegram dan cari bot bernama "@BotFather". Mulai obrolan dengan @BotFather dan jalankan perintah /newbot untuk membuat bot baru. Ikuti petunjuk yang diberikan oleh @BotFather dan dapatkan token bot yang dihasilkan.

## 2. Dapatkan Chat ID
Tambahkan bot Anda ke obrolan grup di Telegram. Kirimkan pesan apa pun ke grup tersebut. Buka URL berikut di web browser Anda: https://api.telegram.org/botYOUR_BOT_TOKEN/getUpdates. Cari "chat" dan temukan "id". Ini adalah ID obrolan (chat ID) yang akan digunakan dalam skrip.

## 3. Kode Python untuk Bot
Simpan skrip berikut sebagai telegram_login_bot.py. Gantilah YOUR_BOT_TOKEN dan YOUR_CHAT_ID dengan token bot dan chat ID yang telah Anda peroleh.
```
import telebot
import subprocess
import time

TOKEN_BOT = 'YOUR_BOT_TOKEN'
CHAT_ID = 'YOUR_CHAT_ID'

bot = telebot.TeleBot(TOKEN_BOT)

@bot.message_handler(commands=['start', 'help'])
def send_welcome(message):
    bot.reply_to(message, "Selamat datang! Saya akan memberi tahu Anda jika ada user yang login ke server.")

def check_login():
    try:
        result = subprocess.check_output("last", shell=True, text=True)
        return result
    except subprocess.CalledProcessError as e:
        return str(e)

def notify_new_login(login_info):
    bot.send_message(CHAT_ID, f"Info Login Baru:\n{login_info}")

if __name__ == "__main__":
    # Dapatkan info login saat skrip pertama kali dijalankan
    previous_logins = check_login()

    while True:
        # Cek info login saat ini
        current_logins = check_login()

        # Bandingkan dengan info login sebelumnya
        if current_logins != previous_logins:
            # Temukan login baru dengan membandingkan info login saat ini dengan sebelumnya
            new_login_info = subprocess.check_output("comm -23 <(echo \"$current_logins\" | cut -d ' ' -f 1-2 | sort) <(echo \"$previous_logins\" | cut -d ' ' -f 1-2 | sort)", shell=True, text=True)

            # Kirim pemberitahuan ke bot Telegram
            notify_new_login(new_login_info)

            # Perbarui info login sebelumnya
            previous_logins = current_logins

        # Tunggu selama 5 menit sebelum memeriksa lagi (sesuaikan sesuai kebutuhan)
        time.sleep(300)

```

## 4. Instalasi Modul Python
Pastikan modul python-telegram-bot terinstal dengan menjalankan perintah:

```
pip install python-telegram-bot
```

## 5. Buat Binary Menggunakan PyInstaller
```
pip install pyinstaller
pyinstaller --onefile telegram_login_bot.py
```
## 6. Letakkan Binary dan Konfigurasi systemd
```
sudo cp dist/telegram_login_bot /usr/local/bin/
sudo cp telegram_login_bot.service /etc/systemd/system/
```
## 7. Berikan Izin yang Tepat
```
sudo chmod +x /usr/local/bin/telegram_login_bot
sudo chmod 644 /etc/systemd/system/telegram_login_bot.service
```
## 8. Restart Daemon systemd
```
sudo systemctl daemon-reload
```
## 9. Mulai dan Aktifkan Layanan
```
sudo systemctl start telegram_login_bot
sudo systemctl enable telegram_login_bot
```
## 10. Periksa Status Layanan
```
sudo systemctl status telegram_login_bot
```

Sekarang, bot Telegram akan memberi tahu Anda setiap kali ada pengguna yang login di server Linux, dan skrip ini akan otomatis berjalan pada sistem startup menggunakan systemd. Jika ada masalah, Anda dapat memeriksa log dengan menjalankan:

```
sudo journalctl -u telegram_login_bot
```



