# 📱 GetContact API Client (Node.js)

[![Node.js Version](https://img.shields.io/badge/node-%3E%3D14.0.0-brightgreen.svg)](https://nodejs.org/)
[![License](https://img.shields.io/badge/license-MIT-blue.svg)](LICENSE)
[![Author](https://img.shields.io/badge/author-gienetic-orange.svg)](https://github.com/giienew)

> Unofficial API Client untuk GetContact berbasis Node.js dengan dukungan penuh untuk enkripsi AES/HMAC dan output JSON murni.

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

1. **Clone repository ini:**
   ```bash
   git clone https://github.com/giienew/getcontact-api-client.git
   cd getcontact-api-client
   ```

2. **Install dependensi:**
   ```bash
   npm install axios
   ```

3. **Verifikasi instalasi:**
   ```bash
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
  USER_AGENT: "Dalvik/2.1.0 (Linux; U; Android 10; Redmi Note 5 Pro Build/QQ3A.200805.001)",
  VALIDATE_PATH: "verify-code"
};
```

### 🔑 Panduan Lengkap Ekstraksi Kredensial

#### **Method 1: Ekstraksi dari Android Device (Recommended)**

**Persyaratan:**
- Device Android dengan root access atau emulator
- ADB (Android Debug Bridge) terinstall

**Langkah-langkah:**

1. **Install GetContact** di device/emulator
2. **Login** ke akun GetContact Anda
3. **Ekstrak preferences file:**

```bash
# Via ADB
adb root
adb pull /data/data/app.source.getcontact/shared_prefs/GetContactSettingsPref.xml

# Via device shell
adb shell
su
cat /data/data/app.source.getcontact/shared_prefs/GetContactSettingsPref.xml
```

4. **Buka file XML** dan cari nilai berikut:
   - `FINAL_KEY` → Gunakan sebagai `AES_KEY_HEX`
   - `TOKEN` → Gunakan sebagai `TOKEN`
   - `DEVICE_ID` → Gunakan sebagai `DEVICE_ID`

**Contoh isi file:**
```xml
<?xml version='1.0' encoding='utf-8' standalone='yes' ?>
<map>
    <string name="FINAL_KEY">a1b2c3d4e5f6...</string>
    <string name="TOKEN">eyJhbGciOiJIUzI1...</string>
    <string name="DEVICE_ID">abc123-device-id</string>
</map>
```

#### **Method 2: Traffic Interception dengan MITM**

**Tools yang dibutuhkan:**
- [Burp Suite](https://portswigger.net/burp) / [HTTP Toolkit](https://httptoolkit.tech/) / [mitmproxy](https://mitmproxy.org/)
- Device Android dengan SSL Pinning bypass (Frida, Xposed)

**Langkah-langkah:**

1. **Setup MITM Proxy** di laptop/PC
2. **Install SSL Certificate** di Android device
3. **Bypass SSL Pinning** menggunakan Frida:
   ```bash
   frida -U -f app.source.getcontact -l ssl-bypass.js --no-pause
   ```
4. **Buka GetContact** dan lakukan pencarian
5. **Intercept request** dan ambil:
   - Header `X-Token` → `TOKEN`
   - Header `X-Client-Device-Id` → `DEVICE_ID`
   - Header `X-Req-Signature` → Untuk reverse HMAC key
   - Request body encrypted → Untuk reverse AES key

#### **Method 3: Reverse Engineering APK**

**Tools:**
- [JADX](https://github.com/skylot/jadx)
- [APKTool](https://ibotpeaches.github.io/Apktool/)
- Frida (untuk dynamic analysis)

**Target:**
- `HMAC_KEY_HEX`: Biasanya hardcoded atau di-obfuscate dalam native library
- Cari di file `.so` di folder `lib/` dalam APK

**Perintah dasar:**
```bash
# Decompile APK
jadx-gui getcontact.apk

# Extract APK
apktool d getcontact.apk

# Analyze native libraries
strings lib/arm64-v8a/libnative.so | grep -i "key\|secret"
```

---

## 📖 Penggunaan

### Basic Usage

```bash
node index.js <NOMOR_TELEPON>
```

### Contoh Lengkap

**Format Indonesia:**
```bash
node index.js 081234567890
node index.js +6281234567890
node index.js 6281234567890
```

**Format Internasional:**
```bash
node index.js +12025550123      # USA
node index.js +447911123456     # UK
node index.js +639171234567     # Philippines
```

### Automasi dengan Script

**Bash script untuk multiple numbers:**
```bash
#!/bin/bash
while IFS= read -r number; do
  echo "Checking: $number"
  node index.js "$number" >> results.json
  sleep 2  # Rate limiting
done < numbers.txt
```

**Python wrapper:**
```python
import subprocess
import json

def lookup_number(phone):
    result = subprocess.run(
        ['node', 'index.js', phone],
        capture_output=True,
        text=True
    )
    return json.loads(result.stdout)

# Usage
info = lookup_number('+6281234567890')
print(f"Name: {info['data']['name']}")
print(f"Tags: {info['data']['tags']}")
```

---

## 🛡️ Penanganan Captcha

### Automatic Captcha Flow

Ketika API memerlukan verifikasi captcha (biasanya setelah beberapa request):

```
┌─────────────────────────────────────────┐
│  1. Request ke API                       │
│  2. Terdeteksi perlu captcha (403004)   │
│  3. Captcha disimpan ke captcha.jpg     │
│  4. User diminta input kode             │
│  5. Validasi kode                        │
│  6. Jika valid → retry request          │
│  7. Jika invalid → ulangi step 3-6      │
└─────────────────────────────────────────┘
```

### Example Output Saat Captcha

```
[!] Validation required (1/5). Image saved: captcha.jpg
Enter captcha code: AB12CD
[+] Validation success, retrying search...

{
  "status": true,
  "meta": {
    "timestamp": "2025-11-28T10:30:45.123Z",
    "phone": "+6281234567890"
  },
  "data": { ... }
}
```

### Tips Menghindari Captcha

- ✅ Gunakan delay antar request (2-5 detik)
- ✅ Jangan melakukan burst requests
- ✅ Gunakan proxy rotation jika melakukan bulk lookup
- ✅ Batasi maksimal 50-100 requests per hari per akun

---

## 📊 Format Output

### ✅ Response Sukses Lengkap

```json
{
  "status": true,
  "meta": {
    "timestamp": "2025-11-28T10:30:45.123Z",
    "phone": "+6282173230348"
  },
  "data": {
    "name": "Echa Putri",
    "tags": [
      "Customer Service",
      "Online Shop",
      "Tokopedia Seller"
    ],
    "tagsCount": 3,
    "usage": {
      "search": {
        "used": 199,
        "limit": 200
      },
      "detail": {
        "used": 145,
        "limit": 150
      }
    },
    "raw": {
      "profile": {
        "displayName": "Echa Putri",
        "phoneNumber": "+6282173230348",
        "countryCode": "62"
      },
      "subscription": {
        "isPremium": false,
        "usage": {
          "search": {
            "remainingCount": 199,
            "limit": 200
          }
        }
      }
    }
  },
  "error": null
}
```

### ❌ Response Error - Rate Limit

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

### ❌ Response Error - Invalid Token

```json
{
  "status": false,
  "meta": {
    "timestamp": "2025-11-28T10:30:45.123Z",
    "phone": "+6281234567890"
  },
  "data": null,
  "error": "401: Unauthorized - Invalid or expired token"
}
```

### ⚠️ Response - Nomor Tidak Ditemukan

```json
{
  "status": true,
  "meta": {
    "timestamp": "2025-11-28T10:30:45.123Z",
    "phone": "+6281234567890"
  },
  "data": {
    "name": null,
    "tags": [],
    "tagsCount": 0,
    "usage": {
      "search": {
        "used": 200,
        "limit": 200
      },
      "detail": null
    },
    "raw": {
      "profile": {},
      "subscription": {}
    }
  },
  "error": null
}
```

---

## 🔧 Troubleshooting

### 🔴 Error: "Invalid AES Key Hex"

**Penyebab:** AES Key tidak valid atau format salah

**Solusi:**
```bash
# Pastikan key dalam format hex (panjang 32, 48, atau 64 karakter)
# Contoh valid: "a1b2c3d4e5f6..." (64 karakter untuk AES-256)
```

### 🔴 Error: "403004: Validation is required"

**Penyebab:** API meminta captcha verification

**Solusi:** Script akan otomatis handle ini:
1. Captcha disimpan sebagai `captcha.jpg`
2. Buka file dan lihat kode
3. Input kode saat diminta
4. Request akan diulang otomatis

### 🔴 Error: "403003: Daily search limit exceeded"

**Penyebab:** Limit harian tercapai (biasanya 200 searches/hari)

**Solusi:**
- Tunggu 24 jam untuk reset
- Gunakan akun berbeda
- Upgrade ke premium (jika tersedia)

### 🔴 Error: "401: Unauthorized"

**Penyebab:** Token expired atau invalid

**Solusi:**
```bash
# 1. Re-extract token dari device
adb pull /data/data/app.source.getcontact/shared_prefs/GetContactSettingsPref.xml

# 2. Update TOKEN di CONFIG
# 3. Pastikan DEVICE_ID juga match
```

### 🔴 Error: "Decrypt Failed"

**Penyebab:** AES Key atau HMAC Key salah

**Solusi:**
- Verifikasi `AES_KEY_HEX` dari field `FINAL_KEY`
- Pastikan tidak ada spasi atau karakter tambahan
- Key harus dalam format hex murni

### 🔴 Rate Limiting / IP Block

**Gejala:** Request selalu timeout atau error 429

**Solusi:**
```bash
# Gunakan proxy
export HTTP_PROXY=http://proxy-server:8080
export HTTPS_PROXY=http://proxy-server:8080
node index.js +6281234567890

# Atau edit axios config di index.js:
const client = axios.create({
  proxy: {
    host: 'proxy-server',
    port: 8080
  },
  // ... config lainnya
});
```

---

## 📁 File Output

Script akan menghasilkan beberapa file debug:

| File | Deskripsi |
|------|-----------|
| `resp_data.b64` | Response terenkripsi dari API (Base64) |
| `resp_data.dec.json` | Response terdekripsi (JSON) |
| `captcha.jpg` | Gambar captcha (jika diminta) |

**Catatan:** File-file ini berguna untuk debugging. Pastikan `.gitignore` sudah mencakup file-file ini.

---

## 🔒 Security Best Practices

### ⚠️ Jangan Hardcode Credentials

**❌ Bad Practice:**
```javascript
const CONFIG = {
  TOKEN: "eyJhbGciOiJIUzI1...",  // Hardcoded!
  AES_KEY_HEX: "a1b2c3d4...",    // Exposed!
  // ...
};
```

**✅ Good Practice:**
```javascript
require('dotenv').config();

const CONFIG = {
  TOKEN: process.env.GETCONTACT_TOKEN,
  AES_KEY_HEX: process.env.GETCONTACT_AES_KEY,
  HMAC_KEY_HEX: process.env.GETCONTACT_HMAC_KEY,
  DEVICE_ID: process.env.GETCONTACT_DEVICE_ID,
  // ...
};
```

**Setup `.env` file:**
```bash
# Install dotenv
npm install dotenv

# Create .env file
cat > .env << EOF
GETCONTACT_TOKEN=your_token_here
GETCONTACT_AES_KEY=your_aes_key_here
GETCONTACT_HMAC_KEY=your_hmac_key_here
GETCONTACT_DEVICE_ID=your_device_id_here
EOF

# Add to .gitignore
echo ".env" >> .gitignore
```

### 🔐 Proteksi Kredensial

```bash
# Enkripsi file config dengan GPG
gpg -c config.json

# Decrypt saat dibutuhkan
gpg -d config.json.gpg > config.json

# Hapus plaintext setelah selesai
rm config.json
```

---

## 🚀 Advanced Usage

### Integrasi dengan Database

**Simpan hasil ke MongoDB:**
```javascript
const mongoose = require('mongoose');
const { exec } = require('child_process');

const ContactSchema = new mongoose.Schema({
  phone: String,
  name: String,
  tags: [String],
  checkedAt: Date
});

const Contact = mongoose.model('Contact', ContactSchema);

function lookupAndSave(phone) {
  exec(`node index.js ${phone}`, (err, stdout) => {
    const result = JSON.parse(stdout);
    if (result.status) {
      Contact.create({
        phone: result.meta.phone,
        name: result.data.name,
        tags: result.data.tags,
        checkedAt: new Date(result.meta.timestamp)
      });
    }
  });
}
```

### REST API Wrapper

**Express.js wrapper:**
```javascript
const express = require('express');
const { exec } = require('child_process');
const app = express();

app.get('/lookup/:phone', (req, res) => {
  const phone = req.params.phone;
  
  exec(`node index.js ${phone}`, (err, stdout) => {
    if (err) {
      return res.status(500).json({ error: err.message });
    }
    res.json(JSON.parse(stdout));
  });
});

app.listen(3000, () => {
  console.log('API running on http://localhost:3000');
});

// Usage: curl http://localhost:3000/lookup/+6281234567890
```

### Telegram Bot Integration

```javascript
const TelegramBot = require('node-telegram-bot-api');
const { exec } = require('child_process');

const bot = new TelegramBot('YOUR_BOT_TOKEN', { polling: true });

bot.onText(/\/lookup (.+)/, (msg, match) => {
  const phone = match[1];
  const chatId = msg.chat.id;
  
  bot.sendMessage(chatId, '🔍 Searching...');
  
  exec(`node index.js ${phone}`, (err, stdout) => {
    const result = JSON.parse(stdout);
    
    if (result.status && result.data.name) {
      bot.sendMessage(chatId, 
        `📱 *${result.meta.phone}*\n` +
        `👤 Name: ${result.data.name}\n` +
        `🏷️ Tags: ${result.data.tags.join(', ') || 'None'}`,
        { parse_mode: 'Markdown' }
      );
    } else {
      bot.sendMessage(chatId, '❌ Number not found');
    }
  });
});
```

---

## 🤝 Contributing

Kontribusi sangat diterima! Berikut cara berkontribusi:

### 1. Fork & Clone

```bash
# Fork repo di GitHub, lalu:
git clone https://github.com/YOUR_USERNAME/getcontact-api-client.git
cd getcontact-api-client
```

### 2. Buat Branch Baru

```bash
git checkout -b feature/awesome-feature
```

### 3. Commit Changes

```bash
git add .
git commit -m "Add: awesome new feature"
```

**Commit message format:**
- `Add:` untuk fitur baru
- `Fix:` untuk bug fixes
- `Update:` untuk improvements
- `Docs:` untuk dokumentasi

### 4. Push & Pull Request

```bash
git push origin feature/awesome-feature
```

Lalu buat Pull Request di GitHub dengan deskripsi lengkap.

### 📋 Contribution Guidelines

- ✅ Ikuti existing code style
- ✅ Tambahkan tests jika memungkinkan
- ✅ Update dokumentasi jika perlu
- ✅ Test secara menyeluruh sebelum PR
- ✅ Satu PR = satu fitur/fix

---

## 📝 Roadmap

### Version 2.0
- [ ] **Batch Lookup** — Support multiple numbers dalam satu run
- [ ] **Rate Limiting** — Auto throttling untuk avoid ban
- [ ] **Proxy Rotation** — Built-in proxy rotation support
- [ ] **Result Caching** — Cache hasil untuk avoid duplicate requests

### Version 2.5
- [ ] **CLI Interface** — Interactive CLI dengan commander.js
- [ ] **Export Formats** — Export ke CSV, Excel, JSON
- [ ] **Config Manager** — GUI untuk manage multiple accounts
- [ ] **Logging System** — Structured logging dengan Winston

### Version 3.0
- [ ] **Docker Support** — Containerized deployment
- [ ] **Web Dashboard** — Simple web UI untuk monitoring
- [ ] **Webhook Support** — Push results ke webhook endpoints
- [ ] **Database Integration** — Built-in support untuk MongoDB/PostgreSQL

### Future Ideas
- [ ] OCR Captcha Solver dengan Tesseract
- [ ] Machine Learning untuk auto tag prediction
- [ ] GraphQL API wrapper
- [ ] Real-time monitoring dashboard
- [ ] Multi-account rotation system

**Vote untuk fitur yang Anda inginkan di [Issues](https://github.com/giienew/getcontact-api-client/issues)!**

---

## ⚠️ Disclaimer

### Legal Notice

Project ini dibuat **untuk tujuan edukasi dan riset protokol aplikasi**.

**SANGAT PENTING - HARAP DIBACA:**

⚠️ **Tentang Privasi:**
- Tool ini dapat mengakses informasi pribadi orang lain
- Penggunaan untuk stalking, harassment, atau tujuan jahat **DILARANG KERAS**
- Hormati privasi orang lain - jangan menyalahgunakan informasi yang didapat

⚠️ **Tentang Legalitas:**
- Penggunaan yang melanggar privasi atau hukum sepenuhnya tanggung jawab pengguna
- Beberapa negara memiliki undang-undang ketat tentang data pribadi (GDPR, dll)
- Pastikan penggunaan Anda comply dengan hukum lokal

⚠️ **Tentang Risiko Teknis:**
- **Account Ban** — Penggunaan berlebihan dapat menyebabkan akun di-banned
- **Rate Limiting** — GetContact membatasi jumlah request per hari
- **IP Block** — Terlalu banyak request dapat memblock IP Anda
- **Captcha** — Frequent usage akan trigger captcha verification
- **API Changes** — GetContact dapat mengubah API tanpa pemberitahuan

⚠️ **Tentang Tool:**
- Tool ini **TIDAK BERAFILIASI** dengan GetContact atau pengembang resminya
- Tidak ada jaminan tool akan terus berfungsi selamanya
- Penulis tidak bertanggung jawab atas kerugian akibat penggunaan tool ini

### Penggunaan yang Diperbolehkan

✅ **Acceptable Use:**
- Verifikasi nomor telepon Anda sendiri
- Riset keamanan dan edukasi
- Checking spam/scam numbers
- Personal use dalam batas wajar

❌ **Prohibited Use:**
- Mass scraping untuk komersial
- Harassment atau stalking
- Selling scraped data
- Violating privacy laws
- Any illegal activities

### Pengakuan

> Dengan menggunakan tool ini, Anda menyatakan bahwa:
> - Anda telah membaca dan memahami disclaimer ini
> - Anda akan menggunakan tool ini secara bertanggung jawab
> - Anda memahami dan menerima semua risiko yang ada
> - Anda tidak akan menyalahgunakan informasi yang didapat

**USE AT YOUR OWN RISK. BE RESPONSIBLE. RESPECT PRIVACY.**

---

## 📄 License

Project ini dilisensikan under **MIT License**.

```
MIT License

Copyright (c) 2025 gienetic

Permission is hereby granted, free of charge, to any person obtaining a copy
of this software and associated documentation files (the "Software"), to deal
in the Software without restriction, including without limitation the rights
to use, copy, modify, merge, publish, distribute, sublicense, and/or sell
copies of the Software, and to permit persons to whom the Software is
furnished to do so, subject to the following conditions:

The above copyright notice and this permission notice shall be included in all
copies or substantial portions of the Software.

THE SOFTWARE IS PROVIDED "AS IS", WITHOUT WARRANTY OF ANY KIND, EXPRESS OR
IMPLIED, INCLUDING BUT NOT LIMITED TO THE WARRANTIES OF MERCHANTABILITY,
FITNESS FOR A PARTICULAR PURPOSE AND NONINFRINGEMENT. IN NO EVENT SHALL THE
AUTHORS OR COPYRIGHT HOLDERS BE LIABLE FOR ANY CLAIM, DAMAGES OR OTHER
LIABILITY, WHETHER IN AN ACTION OF CONTRACT, TORT OR OTHERWISE, ARISING FROM,
OUT OF OR IN CONNECTION WITH THE SOFTWARE OR THE USE OR OTHER DEALINGS IN THE
SOFTWARE.
```

[Full License Text](LICENSE)

---

## 👤 Author

**gienetic**

- 🐙 GitHub: [@giienew](https://github.com/giienew)
- 📧 Issues: [Report Bug](https://github.com/giienew/getcontact-api-client/issues)
- 💡 Discussions: [Feature Requests](https://github.com/giienew/getcontact-api-client/discussions)

---

## 🙏 Acknowledgments

- GetContact team untuk aplikasi yang powerful
- Community contributors yang membantu improve tool ini
- Semua yang memberikan feedback dan bug reports

---

## ⭐ Support This Project

Jika project ini membantu Anda, pertimbangkan untuk:

- ⭐ **Star** repository ini di GitHub
- 🐛 **Report bugs** yang Anda temukan
- 💡 **Suggest features** yang Anda inginkan
- 📖 **Improve documentation**
- 🔀 **Submit Pull Requests**

Setiap bentuk kontribusi sangat dihargai! 🙌

---

## 📊 Project Stats

![GitHub stars](https://img.shields.io/github/stars/giienew/getcontact-api-client?style=social)
![GitHub forks](https://img.shields.io/github/forks/giienew/getcontact-api-client?style=social)
![GitHub issues](https://img.shields.io/github/issues/giienew/getcontact-api-client)
![GitHub pull requests](https://img.shields.io/github/issues-pr/giienew/getcontact-api-client)

---

<div align="center">
  <sub>Built with ❤️ for educational purposes</sub>
  <br>
  <sub>© 2025 gienetic. All rights reserved.</sub>
</div>