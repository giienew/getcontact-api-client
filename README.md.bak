# 📱 GetContact API Client (Node.js)

[![Node.js Version](https://img.shields.io/badge/node-%3E%3D14.0.0-brightgreen.svg)](https://nodejs.org/)
[![License](https://img.shields.io/badge/license-MIT-blue.svg)](LICENSE)
[![NB Community](https://img.shields.io/badge/Released%20by-NB%20Community-red.svg)](https://whatsapp.com/channel/0029Vb5EZCjIiRotHCI1213L)
[![Author](https://img.shields.io/badge/author-gienetic-orange.svg)](https://github.com/giienew)

<div align="center">

### 🔥 OFFICIAL RELEASE BY NB COMMUNITY 🔥

**Developed & Maintained by [gienetic](https://github.com/giienew)**

---

[![NB Community](https://img.shields.io/badge/🚀%20Join%20NB%20Community-WhatsApp-25D366?style=for-the-badge&logo=whatsapp)](https://whatsapp.com/channel/0029Vb5EZCjIiRotHCI1213L)

*Indonesian Elite Network Sniffing, Web Scraping & Reverse Engineering Community*

---

</div>

> **Unofficial API Client untuk GetContact berbasis Node.js** dengan dukungan penuh untuk enkripsi AES/HMAC dan output JSON murni.

Script ini memungkinkan Anda melakukan reverse lookup nomor telepon, memeriksa nama pemilik, melihat tags yang diberikan pengguna lain, serta menangani enkripsi sesuai protokol aplikasi resmi GetContact dengan output JSON yang clean dan siap diintegrasikan.

---

## ✨ Fitur Utama

- 🔍 **Reverse Lookup** — Mencari informasi nama & profil berdasarkan nomor telepon
- 🏷️ **Tags Retrieval** — Mengambil daftar tags yang diberikan pengguna lain
- 🔐 **Auto Encryption/HMAC** — Proses AES-256-ECB & HMAC-SHA256 ditangani otomatis
- 🤖 **Smart Captcha Handling** — Auto-save captcha ke file dan retry otomatis setelah input
- 📊 **Pure JSON Output** — Output bersih tanpa clutter, siap diintegrasikan ke automation
- 🔄 **Auto Retry Logic** — Otomatis mengulang request setelah captcha validation
- 📈 **Usage Tracking** — Monitor limit penggunaan API search & detail
- 🛡️ **Error Handling** — Comprehensive error handling dengan JSON output

---

## 📋 Prasyarat

Sebelum memulai, pastikan sistem Anda memiliki:

- **Node.js** v14 atau lebih tinggi
- **NPM** (biasanya sudah terinstall bersama Node.js)
- **Akses Root** pada device Android (untuk ekstraksi kredensial)

---

## 🚀 Instalasi

```bash
# Clone repository
git clone https://github.com/giienew/getcontact-api-client.git
cd getcontact-api-client

# Install dependensi
npm install axios

# Verifikasi instalasi
node index.js
```

---

## ⚙️ Konfigurasi

Edit bagian `CONFIG` di file `index.js` dengan kredensial akun GetContact Anda:

```javascript
const CONFIG = {
  HOST: "pbssrv-centralevents.com",
  API_VER: "v2.8",
  TOKEN: "YOUR_TOKEN_HERE",              // Required
  AES_KEY_HEX: "YOUR_AES_KEY_HERE",      // Required (FINAL_KEY)
  HMAC_KEY_HEX: "YOUR_HMAC_KEY_HERE",    // Required
  DEVICE_ID: "YOUR_DEVICE_ID",           // Required
  APP_VERSION: "8.8.0",
  OS_STRING: "android 10",
  LANG: "en_US",
  NET_CC: "us",
  USER_AGENT: "gienetic/NB-Team-Community (Linux; U; Android 10; Research Build/NB.2025.001)",
  VALIDATE_PATH: "verify-code"
};
```

### 🔑 Cara Ekstraksi Kredensial

#### **Method 1: Ekstraksi dari Android Device (Recommended)**

**Persyaratan:**
- Device Android dengan root access atau emulator
- ADB (Android Debug Bridge) terinstall

**Langkah-langkah:**

```bash
# Via ADB
adb root
adb pull /data/data/app.source.getcontact/shared_prefs/GetContactSettingsPref.xml

# Via device shell
adb shell
su
cat /data/data/app.source.getcontact/shared_prefs/GetContactSettingsPref.xml
```

Cari nilai berikut di file XML:
- `FINAL_KEY` → Gunakan sebagai `AES_KEY_HEX`
- `TOKEN` → Gunakan sebagai `TOKEN`
- `DEVICE_ID` → Gunakan sebagai `DEVICE_ID`

#### **Method 2: Traffic Interception dengan MITM**

Gunakan tools seperti Burp Suite, HTTP Toolkit, atau mitmproxy untuk intercept traffic dan ambil header `X-Token`, `X-Client-Device-Id`, dan data encrypted untuk reverse engineering.

#### **Method 3: Reverse Engineering APK**

Gunakan JADX atau APKTool untuk decompile APK dan cari key yang di-hardcode di native library.

---

## 📖 Penggunaan

### Basic Usage

```bash
node index.js <NOMOR_TELEPON>
```

### Contoh

```bash
# Format Indonesia
node index.js 081234567890
node index.js +6281234567890

# Format Internasional
node index.js +12025550123      # USA
node index.js +447911123456     # UK
```

### Automasi dengan Script

**Bash script untuk multiple numbers:**
```bash
#!/bin/bash
while IFS= read -r number; do
  echo "Checking: $number"
  node index.js "$number" >> results.json
  sleep 2
done < numbers.txt
```

---

## 🛡️ Penanganan Captcha

Ketika API memerlukan verifikasi captcha, script akan:

1. Menyimpan captcha ke file `captcha.jpg`
2. Meminta input kode dari user
3. Validasi kode
4. Retry request secara otomatis

**Tips menghindari captcha:**
- Gunakan delay 2-5 detik antar request
- Batasi maksimal 50-100 requests per hari
- Jangan melakukan burst requests

---

## 📊 Format Output

### ✅ Response Sukses

```json
{
  "status": true,
  "meta": {
    "timestamp": "2025-11-28T10:30:45.123Z",
    "phone": "+6282173230348"
  },
  "data": {
    "name": "Echa Putri",
    "tags": ["Customer Service", "Online Shop"],
    "tagsCount": 2,
    "usage": {
      "search": { "used": 199, "limit": 200 },
      "detail": { "used": 145, "limit": 150 }
    }
  }
}
```

### ❌ Response Error

```json
{
  "status": false,
  "meta": {
    "timestamp": "2025-11-28T10:30:45.123Z",
    "phone": "+6281234567890"
  },
  "data": null,
  "error": "403003: Daily search limit exceeded"
}
```

---

## 🔧 Troubleshooting

| Error | Penyebab | Solusi |
|-------|----------|--------|
| **Invalid AES Key** | Format key salah | Pastikan key dalam format hex (64 karakter) |
| **403004: Validation required** | Captcha diminta | Input kode dari `captcha.jpg` |
| **403003: Limit exceeded** | Limit harian tercapai | Tunggu 24 jam atau gunakan akun lain |
| **401: Unauthorized** | Token expired | Re-extract token dari device |
| **Decrypt Failed** | AES/HMAC key salah | Verifikasi key dari `FINAL_KEY` |

---

## 🔒 Security Best Practices

**❌ Jangan hardcode credentials:**

```javascript
// Bad
const CONFIG = {
  TOKEN: "NbToken321...",
};
```

**✅ Gunakan environment variables:**

```bash
# Install dotenv
npm install dotenv

# Buat file .env
cat > .env << EOF
GETCONTACT_TOKEN=your_token_here
GETCONTACT_AES_KEY=your_aes_key_here
GETCONTACT_HMAC_KEY=your_hmac_key_here
GETCONTACT_DEVICE_ID=your_device_id_here
EOF

# Tambahkan ke .gitignore
echo ".env" >> .gitignore
```

```javascript
// Good
require('dotenv').config();
const CONFIG = {
  TOKEN: process.env.GETCONTACT_TOKEN,
};
```

---

## 🚀 Advanced Usage

### REST API Wrapper

```javascript
const express = require('express');
const { exec } = require('child_process');
const app = express();

app.get('/lookup/:phone', (req, res) => {
  exec(`node index.js ${req.params.phone}`, (err, stdout) => {
    if (err) return res.status(500).json({ error: err.message });
    res.json(JSON.parse(stdout));
  });
});

app.listen(3000);
```

### Telegram Bot Integration

```javascript
const TelegramBot = require('node-telegram-bot-api');
const { exec } = require('child_process');

const bot = new TelegramBot('YOUR_BOT_TOKEN', { polling: true });

bot.onText(/\/lookup (.+)/, (msg, match) => {
  const phone = match[1];
  bot.sendMessage(msg.chat.id, '🔍 Searching...');
  
  exec(`node index.js ${phone}`, (err, stdout) => {
    const result = JSON.parse(stdout);
    if (result.status && result.data.name) {
      bot.sendMessage(msg.chat.id, 
        `📱 ${result.meta.phone}\n👤 ${result.data.name}\n🏷️ ${result.data.tags.join(', ')}`
      );
    }
  });
});
```

---

## 🤝 Contributing

Kontribusi sangat diterima! Caranya:

```bash
# Fork repo, lalu:
git clone https://github.com/YOUR_USERNAME/getcontact-api-client.git
git checkout -b feature/awesome-feature

# Buat perubahan, lalu:
git commit -m "Add: awesome feature"
git push origin feature/awesome-feature
```

**Commit format:**
- `Add:` untuk fitur baru
- `Fix:` untuk bug fixes
- `Update:` untuk improvements
- `Docs:` untuk dokumentasi

---

## 📝 Roadmap

### Version 2.0
- [ ] Batch lookup untuk multiple numbers
- [ ] Auto rate limiting
- [ ] Proxy rotation support
- [ ] Result caching

### Version 3.0
- [ ] Docker support
- [ ] Web dashboard
- [ ] Database integration
- [ ] OCR captcha solver

---

## ⚠️ Disclaimer

**PENTING - HARAP DIBACA:**

⚠️ Project ini dibuat untuk **tujuan edukasi dan riset** protokol aplikasi.

**Tentang Privasi:**
- Tool ini dapat mengakses informasi pribadi orang lain
- Penggunaan untuk stalking, harassment, atau tujuan jahat **DILARANG KERAS**
- Hormati privasi orang lain

**Tentang Risiko:**
- Account ban jika penggunaan berlebihan
- Rate limiting dan IP block
- GetContact dapat mengubah API kapan saja
- Tool ini **TIDAK BERAFILIASI** dengan GetContact

**Penggunaan yang Diperbolehkan:**
✅ Verifikasi nomor sendiri, riset keamanan, checking spam
❌ Mass scraping, harassment, selling data, illegal activities

**USE AT YOUR OWN RISK. BE RESPONSIBLE. RESPECT PRIVACY.**

---

## 📄 License

Project ini dilisensikan under **MIT License** - lihat file [LICENSE](LICENSE) untuk detail lengkap.

---

## 🏆 Credits & Development Team

<div align="center">

### 🌟 Released & Powered by 🌟

<img src="https://img.shields.io/badge/NB%20COMMUNITY-Official%20Release-red?style=for-the-badge" alt="NB Community">

**NB COMMUNITY**  
*Indonesian Elite Network Sniffing, Web Scraping & Reverse Engineering Community*

Komunitas eksklusif para researcher, developer, dan security enthusiast Indonesia yang fokus pada:
- 🔍 Network Traffic Analysis & Sniffing
- 🕷️ Advanced Web Scraping Techniques  
- 🔐 Protocol Reverse Engineering
- 🛡️ Security Research & Penetration Testing
- 🤖 API Development & Automation

[![Join NB Community](https://img.shields.io/badge/📢%20Join%20Our%20WhatsApp%20Channel-25D366?style=for-the-badge&logo=whatsapp&logoColor=white)](https://whatsapp.com/channel/0029Vb5EZCjIiRotHCI1213L)

---

### 👨‍💻 Lead Developer & Maintainer

**gienetic** - *Lead Developer & Protocol Researcher*

Specialized in protocol reverse engineering, encryption/decryption systems, dan API development. Member aktif NB Community yang berkontribusi dalam riset dan development berbagai tools untuk security research dan automation.

[![GitHub](https://img.shields.io/badge/GitHub-@giienew-181717?style=flat&logo=github)](https://github.com/giienew)
[![WhatsApp](https://img.shields.io/badge/WhatsApp-+62%20895--0547--2234-25D366?style=flat&logo=whatsapp)](https://wa.me/6289505472234)

---

### 🤝 Special Thanks

Terima kasih kepada seluruh member **NB Community** yang telah berkontribusi dalam:
- Knowledge sharing tentang protokol GetContact
- Testing dan bug reporting
- Feedback untuk improvement
- Support moral dan teknis

</div>

---

## 💬 Contact & Support

### 📧 Get in Touch

**For Collaboration & Questions:**
- 👤 Developer: **gienetic**
- 📱 WhatsApp: [+62 895-0547-2234](https://wa.me/6289505472234)
- 🐙 GitHub: [@giienew](https://github.com/giienew)
- 🐛 Report Issues: [GitHub Issues](https://github.com/giienew/getcontact-api-client/issues)
- 💡 Feature Requests: [GitHub Discussions](https://github.com/giienew/getcontact-api-client/discussions)

**Join Our Community:**
- 📢 WhatsApp Channel: [NB Community Official](https://whatsapp.com/channel/0029Vb5EZCjIiRotHCI1213L)
- 🤝 Collaborate, share knowledge, dan belajar bersama para expert!

---

## ⭐ Support This Project

Jika project ini bermanfaat untuk Anda:

- ⭐ **Star** repository ini di GitHub
- 🔔 **Follow** untuk update terbaru
- 📢 **Share** ke teman-teman developer Anda
- 🐛 **Report bugs** yang Anda temukan
- 💡 **Suggest features** melalui Issues
- 🔀 **Submit Pull Requests** untuk improvement

Jangan lupa **join NB Community** untuk:
- 📚 Akses ke exclusive tutorials & resources
- 🎯 Networking dengan expert developers
- 🔥 Early access ke tools & projects terbaru
- 💬 24/7 support dari community members

[![Join Now](https://img.shields.io/badge/🚀%20Join%20NB%20Community%20Now-Click%20Here-red?style=for-the-badge)](https://whatsapp.com/channel/0029Vb5EZCjIiRotHCI1213L)

---

## 📊 Project Stats

![GitHub stars](https://img.shields.io/github/stars/giienew/getcontact-api-client?style=social)
![GitHub forks](https://img.shields.io/github/forks/giienew/getcontact-api-client?style=social)
![GitHub watchers](https://img.shields.io/github/watchers/giienew/getcontact-api-client?style=social)
![GitHub issues](https://img.shields.io/github/issues/giienew/getcontact-api-client)
![GitHub pull requests](https://img.shields.io/github/issues-pr/giienew/getcontact-api-client)

---

<div align="center">

### 🎯 Made with 💙 for Educational & Research Purposes

**Official Release by NB Community**  
Developed & Maintained by **gienetic**

---

*"Knowledge is meant to be shared, security is meant to be improved"*

---

[![NB Community](https://img.shields.io/badge/NB%20Community-Official%20Release-red?style=flat-square)](https://whatsapp.com/channel/0029Vb5EZCjIiRotHCI1213L)
[![Author](https://img.shields.io/badge/Author-gienetic-orange?style=flat-square)](https://github.com/giienew)
[![License](https://img.shields.io/badge/License-MIT-blue?style=flat-square)](LICENSE)

**© 2025 NB Community & gienetic. All rights reserved.**

</div>