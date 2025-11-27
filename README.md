Siap, ini **versi full text langsung siap copas** tanpa blok kode, tanpa pemisah — tinggal tempel di README GitHub.

---

# GetContact API Client (Node.js)

Unofficial API Client untuk GetContact berbasis Node.js. Script ini memungkinkan Anda melakukan reverse lookup, memeriksa nama pemilik, melihat tags, serta menangani enkripsi AES/HMAC sesuai protokol aplikasi resmi. Output sepenuhnya JSON murni sehingga mudah diintegrasikan ke automation, bot, atau aplikasi lainnya.

## ✨ Fitur

* Reverse Lookup — Mencari informasi nama & profil berdasarkan nomor HP.
* Tags Retrieval — Mengambil daftar tags yang diberikan pengguna lain.
* Auto Encryption/HMAC — Proses AES & HMAC ditangani otomatis.
* Captcha Handling — Menyimpan captcha ke file, meminta input, lalu mengulang request.
* JSON Output — Bersih dan siap dipakai untuk pipeline.

## 📦 Prasyarat

* Node.js v14+
* NPM
* Dependensi tambahan: axios
* Modul bawaan: crypto, fs, readline

## 🚀 Instalasi

1. Clone repo:
   git clone [https://github.com/giienew/getcontact-api-client.git](https://github.com/giienew/getcontact-api-client.git)
   cd repo-ini
2. Install dependensi:
   npm install axios

## ⚙️ Konfigurasi

Edit bagian CONFIG di file index.js sesuai data akun Anda:

const CONFIG = {
HOST: "pbssrv-centralevents.com",
API_VER: "v2.8",
TOKEN: "# Token",
AES_KEY_HEX: "# AES Key (Final_Key)",
HMAC_KEY_HEX: "# HMAC key",
DEVICE_ID: "Id Device Kamu",
APP_VERSION: "8.8.0",
OS_STRING: "android 10",
LANG: "en_US",
NET_CC: "us",
USER_AGENT: "Dalvik/2.1.0 (Linux; U; Android 10; Redmi Note 5 Pro Build/QQ3A.200805.001)",
VALIDATE_PATH: "verify-code"
};

### Cara mendapatkan token & key

* AES Key & Token: /data/data/app.source.getcontact/shared_prefs/GetContactSettingsPref.xml (field: FINAL_KEY, TOKEN)
* HMAC Key: hasil dekripsi fungsi signing aplikasi (static/dynamic analysis)
* Bisa juga melalui sniffing trafik memakai Burp, HTTP Toolkit, atau mitm proxy lain

## 📖 Penggunaan

Jalankan command:
node index.js 081234567890

## 🛡️ Penanganan Captcha

Jika API meminta verifikasi:
• Script menyimpan file captcha.jpg
• Terminal meminta input captcha
• Setelah benar, request otomatis diulang

## 📄 Contoh Output (Sukses)

{
"status": true,
"meta": {
"timestamp": "2025-11-27T22:42:30.801Z",
"phone": "+6282173230348"
},
"data": {
"name": "Echa Putri",
"tags": [],
"tagsCount": 0,
"usage": {
"search": { "used": 199, "limit": 200 }
},
"raw": { ... }
},
"error": null
}

## ⚠️ Disclaimer

Project ini dibuat untuk tujuan edukasi dan riset protokol aplikasi. Penggunaan yang melanggar privasi atau hukum sepenuhnya menjadi tanggung jawab pengguna. Risiko seperti banned akun, limit API, captcha, atau pemblokiran IP bukan tanggung jawab penulis. Gunakan dengan bijak.

---
