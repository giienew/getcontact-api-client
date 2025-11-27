# 📱 GetContact API Client (Node.js)

[![Node.js Version](https://img.shields.io/badge/node-%3E%3D14.0.0-brightgreen.svg)](https://nodejs.org/)
[![License](https://img.shields.io/badge/license-MIT-blue.svg)](LICENSE)

> Unofficial API Client untuk GetContact berbasis Node.js dengan dukungan penuh untuk enkripsi AES/HMAC dan output JSON.

Script ini memungkinkan Anda melakukan reverse lookup nomor telepon, memeriksa nama pemilik, melihat tags yang diberikan pengguna lain, serta menangani enkripsi sesuai protokol aplikasi resmi GetContact.

---

## ✨ Fitur

- 🔍 **Reverse Lookup** — Mencari informasi nama & profil berdasarkan nomor telepon
- 🏷️ **Tags Retrieval** — Mengambil daftar tags yang diberikan pengguna lain
- 🔐 **Auto Encryption/HMAC** — Proses AES & HMAC ditangani otomatis
- 🤖 **Captcha Handling** — Menyimpan captcha ke file, meminta input, lalu mengulang request
- 📊 **JSON Output** — Output bersih dan siap diintegrasikan ke automation, bot, atau aplikasi lainnya

---

## 📋 Prasyarat

Sebelum memulai, pastikan sistem Anda memiliki:

- **Node.js** v14 atau lebih tinggi
- **NPM** (biasanya sudah terinstall bersama Node.js)

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

---

## ⚙️ Konfigurasi

Edit bagian `CONFIG` di file `index.js` sesuai dengan data akun GetContact Anda:

```javascript
const CONFIG = {
  HOST: "pbssrv-centralevents.com",
  API_VER: "v2.8",
  TOKEN: "YOUR_TOKEN_HERE",
  AES_KEY_HEX: "YOUR_AES_KEY_HERE",
  HMAC_KEY_HEX: "YOUR_HMAC_KEY_HERE",
  DEVICE_ID: "YOUR_DEVICE_ID",
  APP_VERSION: "8.8.0",
  OS_STRING: "android 10",
  LANG: "en_US",
  NET_CC: "us",
  USER_AGENT: "Dalvik/2.1.0 (Linux; U; Android 10; Redmi Note 5 Pro Build/QQ3A.200805.001)",
  VALIDATE_PATH: "verify-code"
};
```

### 🔑 Cara Mendapatkan Token & Key

Terdapat beberapa metode untuk mendapatkan kredensial yang diperlukan:

#### **Metode 1: Dari File Preferences Android**
```bash
/data/data/app.source.getcontact/shared_prefs/GetContactSettingsPref.xml
```
- **AES Key**: Cari field `FINAL_KEY`
- **Token**: Cari field `TOKEN`

#### **Metode 2: Traffic Sniffing**
Gunakan salah satu tools berikut untuk intercept HTTP traffic:
- Burp Suite
- HTTP Toolkit
- mitmproxy
- Charles Proxy

#### **Metode 3: Reverse Engineering**
- **HMAC Key**: Hasil dekripsi fungsi signing aplikasi (memerlukan static/dynamic analysis)

> ⚠️ **Catatan**: Proses ekstraksi memerlukan akses root pada perangkat Android atau emulator.

---

## 📖 Penggunaan

Jalankan script dengan format:

```bash
node index.js <NOMOR_TELEPON>
```

**Contoh:**
```bash
node index.js 081234567890
```

atau dengan kode negara:

```bash
node index.js +6281234567890
```

---

## 🛡️ Penanganan Captcha

Jika API memerlukan verifikasi captcha:

1. Script otomatis menyimpan captcha sebagai `captcha.jpg`
2. Terminal akan meminta input kode captcha
3. Masukkan kode yang terlihat pada gambar
4. Request akan diulang secara otomatis setelah verifikasi berhasil

**Contoh Flow:**
```
❌ Captcha required!
💾 Captcha saved to: captcha.jpg
🔍 Please solve the captcha and enter the code: 
> AB12CD
✅ Retrying with captcha...
```

---

## 📊 Contoh Output

### ✅ Response Sukses

```json
{
  "status": true,
  "meta": {
    "timestamp": "2025-11-27T22:42:30.801Z",
    "phone": "+6282173230348"
  },
  "data": {
    "name": "Echa Putri",
    "tags": ["Customer Service", "Online Shop"],
    "tagsCount": 2,
    "usage": {
      "search": { 
        "used": 199, 
        "limit": 200 
      }
    },
    "raw": { 
      // Data lengkap dari API
    }
  },
  "error": null
}
```

### ❌ Response Error

```json
{
  "status": false,
  "meta": {
    "timestamp": "2025-11-27T22:42:30.801Z",
    "phone": "+6282173230348"
  },
  "data": null,
  "error": {
    "code": "RATE_LIMIT",
    "message": "Daily search limit exceeded"
  }
}
```

---

## 🔧 Troubleshooting

### Problem: "Token expired" atau "Unauthorized"
**Solusi**: Update `TOKEN` di konfigurasi dengan token terbaru dari aplikasi.

### Problem: Captcha terus muncul
**Solusi**: 
- Gunakan proxy yang berbeda
- Tunggu beberapa menit sebelum mencoba lagi
- Pastikan device fingerprint valid

### Problem: "Rate limit exceeded"
**Solusi**: GetContact membatasi jumlah pencarian per hari. Tunggu hingga limit reset (biasanya 24 jam).

---

## 🤝 Kontribusi

Kontribusi selalu diterima! Jika Anda ingin berkontribusi:

1. Fork repository ini
2. Buat branch fitur baru (`git checkout -b feature/AmazingFeature`)
3. Commit perubahan (`git commit -m 'Add some AmazingFeature'`)
4. Push ke branch (`git push origin feature/AmazingFeature`)
5. Buat Pull Request

---

## 📝 Roadmap

- [ ] Support untuk batch lookup (multiple numbers)
- [ ] Implementasi rate limiting otomatis
- [ ] Proxy rotation support
- [ ] CLI interface yang lebih interaktif
- [ ] Export hasil ke CSV/Excel
- [ ] Docker containerization

---

## ⚠️ Disclaimer

Project ini dibuat **untuk tujuan edukasi dan riset protokol aplikasi**. 

**Harap diperhatikan:**
- Penggunaan yang melanggar privasi atau hukum sepenuhnya menjadi tanggung jawab pengguna
- Risiko seperti banned akun, limit API, captcha berkelanjutan, atau pemblokiran IP bukan tanggung jawab pengembang
- Script ini tidak berafiliasi dengan GetContact atau pengembang resminya
- Gunakan dengan bijak dan hormati privasi orang lain

---

## 📄 License

Project ini dilisensikan under [MIT License](LICENSE) - lihat file LICENSE untuk detail lebih lanjut.

---

## 👤 Author

**giienew**
- GitHub: [@giienew](https://github.com/giienew)

---

## ⭐ Support

Jika project ini membantu Anda, berikan ⭐ di repository ini!

---

<div align="center">
  <sub>Built with ❤️ for educational purposes</sub>
</div>