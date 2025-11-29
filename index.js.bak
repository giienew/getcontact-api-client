/**
 * ------------------------------------------------------------
 *  GetContact API Client (Node.js)
 *  Author      : gienetic
 *  Description : Reverse lookup & tag scraping with full encryption support.
 * ------------------------------------------------------------
 */

const crypto = require('crypto');
const fs = require('fs');
const readline = require('readline');
const axios = require('axios');

const CONFIG = {
  HOST: "pbssrv-centralevents.com",
  API_VER: "v2.8",
  TOKEN: "",
  AES_KEY_HEX: "",
  HMAC_KEY_HEX: "",
  DEVICE_ID: "",
  APP_VERSION: "8.8.0",
  OS_STRING: "android 10",
  LANG: "en_US",
  NET_CC: "us",
  USER_AGENT: "Dalvik/2.1.0 (Linux; U; Android 10; Redmi Note 5 Pro Build/QQ3A.200805.001)",
  VALIDATE_PATH: "verify-code"
};

class AESCipher {
  constructor(keyHex) {
    try {
      this.key = Buffer.from(keyHex, 'hex');
    } catch (e) {
      throw new Error("Invalid AES Key Hex");
    }
    if (this.key.length === 32) this.alg = 'aes-256-ecb';
    else if (this.key.length === 16) this.alg = 'aes-128-ecb';
    else if (this.key.length === 24) this.alg = 'aes-192-ecb';
    else throw new Error(`Invalid AES Key length: ${this.key.length}`);
    this.blockSize = 16;
  }

  _pad(buffer) {
    const pad = this.blockSize - (buffer.length % this.blockSize);
    const padding = Buffer.alloc(pad, pad);
    return Buffer.concat([buffer, padding]);
  }

  _unpad(buffer) {
    if (buffer.length === 0) return buffer;
    const pad = buffer[buffer.length - 1];
    if (pad < 1 || pad > 16) return buffer;
    return buffer.slice(0, buffer.length - pad);
  }

  encryptBase64(jsonStr) {
    const pt = this._pad(Buffer.from(jsonStr, 'utf8'));
    const cipher = crypto.createCipheriv(this.alg, this.key, null);
    cipher.setAutoPadding(false);
    let ct = Buffer.concat([cipher.update(pt), cipher.final()]);
    return ct.toString('base64');
  }

  decryptBase64(encBase64) {
    let b64 = encBase64;
    while (b64.length % 4 !== 0) b64 += "=";
    const ct = Buffer.from(b64, 'base64');
    const decipher = crypto.createDecipheriv(this.alg, this.key, null);
    decipher.setAutoPadding(false);
    let pt = Buffer.concat([decipher.update(ct), decipher.final()]);
    return this._unpad(pt).toString('utf8');
  }
}

const aes = new AESCipher(CONFIG.AES_KEY_HEX);
let hmacKeyBuffer;
try {
  hmacKeyBuffer = Buffer.from(CONFIG.HMAC_KEY_HEX, 'hex');
} catch (e) {
  hmacKeyBuffer = Buffer.from(CONFIG.HMAC_KEY_HEX, 'utf8');
}

let _lastTs = 0;
function getTimestamp() {
  let t = Date.now();
  if (t <= _lastTs) t = _lastTs + 1;
  _lastTs = t;
  return t.toString();
}

function signRequest(ts, obj) {
  const bodyStr = JSON.stringify(obj);
  const msg = `${ts}-${bodyStr}`;
  const hmac = crypto.createHmac('sha256', hmacKeyBuffer);
  hmac.update(msg, 'utf8');
  const sig = hmac.digest('base64');
  return { signature: sig, bodyStr: bodyStr };
}

const client = axios.create({
  headers: {
    "User-Agent": CONFIG.USER_AGENT,
    "X-Os": CONFIG.OS_STRING,
    "X-Mobile-Service": "GMS",
    "X-Store": "PLAYSTORE",
    "X-App-Version": CONFIG.APP_VERSION,
    "X-Client-Device-Id": CONFIG.DEVICE_ID,
    "X-Lang": CONFIG.LANG,
    "X-Token": CONFIG.TOKEN,
    "X-Encrypted": "1",
    "X-Roaming-Country": CONFIG.NET_CC,
    "X-Network-Country": CONFIG.NET_CC,
    "X-Country-Code": CONFIG.NET_CC,
    "Content-Type": "application/json; charset=utf-8",
    "Accept-Encoding": "gzip, deflate, br",
  },
  timeout: 25000
});

const rl = readline.createInterface({
  input: process.stdin,
  output: process.stderr 
});

const askQuestion = (query) => new Promise((resolve) => rl.question(query, resolve));

async function sendPost(endpoint, bodyObj, pathHint) {
  const ts = getTimestamp();
  const { signature, bodyStr } = signRequest(ts, bodyObj);
  const encBase64 = aes.encryptBase64(bodyStr);
  const url = `https://${CONFIG.HOST}/${CONFIG.API_VER}/${endpoint}`;
  const finalPayload = { data: encBase64 };
  const headers = {
    "X-Req-Timestamp": ts,
    "X-Req-Signature": signature,
    "X-Path": `[${CONFIG.API_VER}, ${pathHint}]`
  };

  try {
    const response = await client.post(url, finalPayload, { headers });
    return processResponse(response);
  } catch (error) {
    if (error.response && error.response.data) {
      return processResponse(error.response);
    }
    throw error;
  }
}

function processResponse(response) {
  const respData = response.data;
  if (!respData || !respData.data) throw new Error(`${response.status} No 'data' field`);
  
  const enc = respData.data;
  fs.writeFileSync("resp_data.b64", enc);
  
  let dec = "";
  try {
    dec = aes.decryptBase64(enc);
    fs.writeFileSync("resp_data.dec.json", dec);
  } catch (e) {
    throw new Error(`Decrypt Failed: ${e.message}`);
  }

  let jsonResult;
  try {
    jsonResult = JSON.parse(dec);
  } catch (e) {
    throw new Error(`Invalid JSON after decrypt: ${dec}`);
  }

  if (response.status !== 200) {
    let msgShort = dec;
    if (jsonResult.meta && jsonResult.meta.errorMessage) {
      msgShort = `${jsonResult.meta.errorCode}: ${jsonResult.meta.errorMessage}`;
      const err = new Error(msgShort);
      err.jsonResult = jsonResult; 
      throw err;
    }
    throw new Error(`${response.status} API Error: ${msgShort}`);
  }
  return jsonResult;
}

function dialCc(e164) {
  const s = e164.replace(/^\+/, '');
  if (s.startsWith('7')) return '7';
  if (s.startsWith('1')) return '1';
  if (s.startsWith('63')) return '63';
  return s.substring(0, 1) || "7";
}

async function searchNumber(phone) {
  const body = {
    "countryCode": dialCc(phone),
    "dn": null,
    "dualSim": null,
    "esim": false,
    "inC": false,
    "phoneNumber": phone,
    "source": "searchHistory",
    "token": CONFIG.TOKEN
  };
  return await sendPost("search", body, "search");
}

async function numberDetail(phone) {
  const body = {
    "countryCode": dialCc(phone),
    "source": "details",
    "token": CONFIG.TOKEN,
    "phoneNumber": phone
  };
  return await sendPost("number-detail", body, "number-detail");
}

async function validateUser(code) {
  const body = { "validationCode": code, "token": CONFIG.TOKEN };
  return await sendPost(CONFIG.VALIDATE_PATH, body, "verify-code");
}

function saveCaptchaImage(jsonResult, outPath = "captcha.jpg") {
  try {
    const imgB64 = jsonResult?.result?.image;
    if (!imgB64) return false;
    const cleanB64 = imgB64.replace(/\\\//g, '/');
    const buffer = Buffer.from(cleanB64, 'base64');
    fs.writeFileSync(outPath, buffer);
    return true;
  } catch (e) {
    return false;
  }
}

async function solveAndValidate(phone, maxAttempts = 5) {
  for (let i = 1; i <= maxAttempts; i++) {
    let errorJson;
    try {
      const content = fs.readFileSync("resp_data.dec.json", "utf8");
      errorJson = JSON.parse(content);
    } catch (e) {
      throw new Error("Captcha payload missing");
    }

    if (!saveCaptchaImage(errorJson)) throw new Error("Failed to extract captcha image");

    process.stderr.write(`\n[!] Validation required (${i}/${maxAttempts}). Image saved: captcha.jpg\n`);
    const code = await askQuestion("Enter captcha code: ");

    try {
      await validateUser(code.trim());
      return;
    } catch (e) {
      const msg = e.message || "";
      if (msg.includes("403004") || msg.includes("invalid")) {
        process.stderr.write("[-] Invalid code, retrying...\n");
        try {
          await searchNumber(phone);
        } catch (se) {
          if (se.message.includes("403004")) continue;
          throw se;
        }
      } else {
        throw e;
      }
    }
  }
  throw new Error("Validation failed: max attempts reached");
}

function pickFirstResult(resp) {
  const res = resp.result;
  if (!res) return [{}, {}, null, res];
  if (!Array.isArray(res)) return [res.profile || {}, res.subscriptionInfo || {}, res.tags, res];
  if (res.length > 0) {
    const item = res[0] || {};
    return [item.profile || item, item.subscriptionInfo || {}, item.tags, item];
  }
  return [{}, {}, null, res];
}

function getName(profile) {
  const keys = ["displayName", "name", "title", "label"];
  for (const k of keys) {
    if (profile[k]) return profile[k];
  }
  return null;
}

function fmtUsage(usage, sec) {
  if (!usage) return null;
  const r = usage[sec] || {};
  return { used: r.remainingCount, limit: r.limit };
}

async function main() {
  const args = process.argv.slice(2);
  const output = {
    status: false,
    meta: {
      timestamp: new Date().toISOString(),
      phone: null,
    },
    data: null,
    error: null
  };

  if (args.length < 1) {
    output.error = "Usage: node index.js <phone_number>";
    console.log(JSON.stringify(output, null, 2));
    process.exit(1);
  }

  let phone = args[0].trim();
  phone = phone.replace(/[^0-9+]/g, '');
  if (phone.startsWith('00')) phone = '+' + phone.slice(2);
  if (!phone.startsWith('+')) phone = '+' + phone;

  output.meta.phone = phone;

  let sr;
  try {
    sr = await searchNumber(phone);
  } catch (e) {
    const msg = e.message;
    if (msg.includes("403004") || msg.includes("validation is required")) {
      try {
        await solveAndValidate(phone);
        process.stderr.write("[+] Validation success, retrying search...\n");
        sr = await searchNumber(phone);
      } catch (e2) {
        output.error = e2.message;
        console.log(JSON.stringify(output, null, 2));
        process.exit(2);
      }
    } else {
      output.error = msg;
      console.log(JSON.stringify(output, null, 2));
      process.exit(2);
    }
  }

  const [p, sub, tagsInline] = pickFirstResult(sr);
  let name = getName(p);
  let tags = null;
  let usage = {};

  try {
    const dt = await numberDetail(phone);
    const [p2, s2, tagsRes] = pickFirstResult(dt);
    if (!name) {
      const n2 = getName(p2);
      if (n2) name = n2;
    }
    usage = (s2 && s2.usage) || (sub && sub.usage) || {};
    tags = tagsRes;
  } catch (e) {
    usage = (sub && sub.usage) || {};
    tags = tagsInline;
  }

  const finalTags = (Array.isArray(tags)) ? tags.map(t => (typeof t === 'object' && t.tag) ? t.tag : t) : [];

  output.status = true;
  output.data = {
    name: name || null,
    tags: finalTags,
    tagsCount: finalTags.length,
    usage: {
      search: fmtUsage(usage, 'search'),
      detail: fmtUsage(usage, 'numberDetail')
    },
    raw: {
      profile: p,
      subscription: sub
    }
  };

  console.log(JSON.stringify(output, null, 2));
  rl.close();
}

main();