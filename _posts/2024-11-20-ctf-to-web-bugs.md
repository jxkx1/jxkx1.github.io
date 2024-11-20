---
layout: post
title: "From CTF's to Web Bugs"
subtitle: "Discovering My First Valid Vulnerability"
author: "jxkxl"
header-style: text
tags:
  - Post
  - Web
  - Bug Hunting
---

As a cybersecurity enthusiast, my primary focus has always been on digital forensics, with a side of binary exploitation and kernel hacking. The thrill of breaking down complex low-level systems and manipulating them to behave unexpectedly has been my passion. Web security? Not so much. My experience in web testing was limited to solving a few beginner-level challenges on platforms like TryHackMe and some Capture the Flag (CTF) challenges. I’ve never considered myself a web expert, nor have I ever really dived into the intricacies of web applications.  

So, imagine my surprise when, during a routine interaction with a web application, I stumbled across what turned out to be my first valid bug. It wasn’t the ultra-critical, headline-worthy discovery that hackers dream of, but it was still significant. And for me, it was a gateway into the fascinating world of web application security.  

Here’s how it all happened.  

---

## A Suspicious URL  

The whole thing started when I was waiting for a 2FA code. The login process had taken me to a new page, and the URL immediately caught my eye. It was long, messy, and packed with encoded data. Something like this:  

```  
https://REDACTED/f34c9bc0-3018-44e8-bb81-ddb4baccc37e/B2C_1_Signup_Signin_PROD/api/CombinedSigninAndSignup/confirmed?
rememberMe=false&csrf_token=MU5ueEVvclR2NDVsM2xqc2VKV01JSTdJWFFXVkhJMm1oQmhOTE9neG05UGdMUkZTQktyYm51RkErRDZON0ZGRFRJZ1NKRVM1K04vaWdnMG10MVRaY0E9PTsyMDI0LTExLTIwVDIwOjI0OjE5LjQ2Mzc5NzhaO3lURFpOak5qMlJvV3VtZHZzcXJLZkE9PTt7Ik9yY2hlc3RyYXRpb25TdGVwIjoxfQ==
&tx=StateProperties=eyJUSUQiOiI5NmI4Y2U3YS01ZmZkLTQwYTctYjU2Ny1lM2EwOGNkZTNkZjkifQ&p=B2C_1_Signup_Signin_PROD
&diags=%7B%22pageViewId%22%3A%2295f2fc25-7d2a-44b1-8971-5cf9aee4f47f%22%2C%22pageId%22%3A%22CombinedSigninAndSignup
%22%2C%22trace%22%3A%5B%7B%22ac%22%3A%22T005%22%2C%22acST%22%3A1732134256%2C%22acD%22%3A1%7D%2C%7B%22ac%22%3A%22T021...
```  

Now, most people would ignore something like this. But I couldn’t resist digging in—this is what years of analyzing encoded data does to you. I copied the entire URL into CyberChef to see what secrets it held.  

---

## Step 1: Decoding the CSRF Token  

The first thing I noticed was a parameter named `csrf_token`. Cross-Site Request Forgery (CSRF) tokens are meant to protect web applications from malicious requests, but here it was, sitting openly in the URL for anyone to see.  

Curious, I decoded the token and found more than just a random string. It included a timestamp, orchestration steps, and what appeared to be user session data:  

```json
{
  "csrf_token": "MU5ueEVvclR2NDVsM2xqc2VKV01JSTdJWFFXVkhJMm1oQmhOTE9neG05UGdMUkZTQktyYm51RkErRDZON0ZGRFRJZ1NKRVM1K04vaWdnMG10MVRaY0E9PTsyMDI0LTExLTIwVDIwOjI0OjE5LjQ2Mzc5NzhaO3lURFpOak5qMlJvV3VtZHZzcXJLZkE9PTt7Ik9yY2hlc3RyYXRpb25TdGVwIjoxfQ==",
  "timestamp": "2024-11-20T20:24:19.4637978Z",
  "orchestrationStep": 1
}
```  

This blew my mind. CSRF tokens are supposed to protect users from malicious requests, but here it was, exposed in the URL. Along with the token itself, there was a timestamp and a reference to the orchestration step of the authentication process.  

This was my first red flag. If I could see this token, so could anyone else with access to the URL, such as via browser history or a phishing attempt. That meant an attacker could potentially hijack a user’s session and perform unauthorized actions on their behalf.

---

## Step 2: A Public Azure Blob  

Encouraged by this discovery, I turned my attention to other parts of the URL. One particular section caught my eye:  

```  
"ac": "T021 - URL:https://REDACTED/prd-userflow/REDACTED_B2C_SignIn_PRD.html"
```  

This appeared to be an Azure Blob Storage URL. Out of curiosity, I navigated to it in my browser. To my surprise, the page loaded without requiring any authentication.  

What I found there were files directly tied to the authentication flow. While I won’t share the specific contents for security reasons, it’s worth mentioning the kinds of data typically stored in such blobs:  
- **Authentication Workflows**: Scripts or templates used to process user logins and registrations.  
- **User Flow Metadata**: Configuration data describing how users interact with the system.  
- **Debugging Artifacts**: Logs or debug files left behind during development.  

This was a classic case of misconfigured cloud storage. Exposing such resources allows attackers to enumerate files, learn about the application’s structure, for an attacker, this is a goldmine. Publicly accessible blob storage could enable resource enumeration, giving malicious actors insights into the system and potentially aiding in crafting attacks.

---

## Step 3: Diagnostic Data in Plain View  

At this point, I was hooked. I went back to the URL and noticed another parameter labeled diag. It appeared to hold diagnostic data about the web application. Upon decoding, I found internal traces and metadata:

```json
{
  "pageViewId": "95f2fc25-7d2a-44b1-8971-5cf9aee4f47f",
  "pageId": "CombinedSigninAndSignup",
  "trace": [
    {"ac": "T005", "acST": 1732134256, "acD": 1},
    {"ac": "T021", "acST": 1732134256, "acD": 265},
    {"ac": "T004", "acST": 1732134256, "acD": 1}
  ]
}
```  

This data, meant for debugging purposes, included traces of how the application processed authentication requests. While this might not seem critical on its own, attackers could use it to gain insight into the application’s inner workings, potentially identifying other vulnerabilities.  

---

## The Bigger Picture  

Reflecting on this discovery, I realized the vulnerabilities were not hyper-critical but still significant:  
1. **CSRF Token Leakage**: Exposed tokens in URLs undermine their purpose and put user sessions at risk.  
2. **Public Azure Blob Storage**: Misconfigured blob storage can expose sensitive resources, aiding attackers.  
3. **Exposed Diagnostic Data**: Internal traces and logs provide attackers with valuable insights into the application.  

For me, this was a groundbreaking moment. I wasn’t actively searching for vulnerabilities, nor did I have extensive experience in web application security. I had stumbled upon these issues out of curiosity while waiting for a 2FA code.  

---

## Lessons Learned  

This experience taught me several important lessons:  
- Always be curious. Even if you’re not an expert in a particular field, your instincts can guide you toward interesting discoveries.  
- Developers should **never expose sensitive data** like CSRF tokens in URLs or leave diagnostic data accessible in production environments.  
- Properly configure cloud storage. Misconfigured resources remain one of the most common vulnerabilities in modern applications.  

For a guy like me, this was an unexpected and rewarding dive into the world of web security. It’s amazing how much you can uncover with just a curious mind and the right tools.  

## Thank you for reading :)