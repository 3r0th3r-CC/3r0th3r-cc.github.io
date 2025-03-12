---
title: VishwaCTF 2025 Writeup
description: "VishwaCTF is the Flagship event of CyberCell-VIIT. It will be an Jeopardy style CTF with team participation. The CTF will be live for 48 hours, and top winners will be awarded with prizes. A beginer friendly CTF open for all"
cover: >-
  https://raw.githubusercontent.com/3r0th3r-CC/3r0th3r-CC.github.io/master/source/assets/images/posts/VishwaCTF-2025/thumb.png
categories:
  - [CTF, Writeups]
tags:
  - web
  - forensics
  - reverse
  - crypto
  - misc
  - osint
  - stegano
  - network
indexing: true
comments: true
date: 2025-03-11 16:00:19
---

Hi, recently, we solved **21 challenges** and got **8th place** in **VishwaCTF**. Since the organizer requires a writeup, we decided to publish it on our blog as well. Enjoy! <(￣︶￣)>

# Miscellaneous

## Sanity Check

![SanityCheck](https://raw.githubusercontent.com/3r0th3r-CC/3r0th3r-CC.github.io/master/source/assets/images/posts/VishwaCTF-2025/Misc/sanity-check.png)

There is a `sanity.txt` and if you look carefully, you will see something like **Vish...**. I just need to use `gedit` to zoom it out and I got the flag

![flag](https://raw.githubusercontent.com/3r0th3r-CC/3r0th3r-CC.github.io/master/source/assets/images/posts/VishwaCTF-2025/Misc/flag.png)

> **FLAG: VishwaCTF{being_a_jack_of_all_is_better_then_being_a_master_at_one}**

# Web Exploitation

## Flames

![Flames](https://raw.githubusercontent.com/3r0th3r-CC/3r0th3r-CC.github.io/master/source/assets/images/posts/VishwaCTF-2025/Web/flames.png)

![web](https://raw.githubusercontent.com/3r0th3r-CC/3r0th3r-CC.github.io/master/source/assets/images/posts/VishwaCTF-2025/Web/flames-web.png)

First, I look at the challenge and its source. I noticed that it required a username and password and it also has a term **Query of Love**, which immediately reminded me of the SQL Injection

I tried to `'OR 1=1;--` for both the username and password fields, but it didn't work

Then, I tried to break the algorithm of the flame calculator and still got nothing (-_-)

The only idea is from the query, maybe it was SQL Injection but with different payload. I tried `'UNION SELECT 1;--` and a new page show up

![famouslovestory](https://raw.githubusercontent.com/3r0th3r-CC/3r0th3r-CC.github.io/master/source/assets/images/posts/VishwaCTF-2025/Web/flames-famous-love-story.png)

Finally, I just need to follow https://flames.ctf-delivery-net.pp.ua/lovers_db.php in **Famous Love Stories** to get the flag :3

![flag](https://raw.githubusercontent.com/3r0th3r-CC/3r0th3r-CC.github.io/master/source/assets/images/posts/VishwaCTF-2025/Web/flames-flag.png)

> **FLAG: VishwaCTF{SQL_1nj3ct10n_C4n_Qu3ry_Your_He4rt}**

P/s: I realized the challenge is actually very easy, any payload with `UNION SELECT` will work lmao

## scan-it-to-stay-safe

![scan-it-to-stay-safe](https://raw.githubusercontent.com/3r0th3r-CC/3r0th3r-CC.github.io/master/source/assets/images/posts/VishwaCTF-2025/Web/scan-it-to-stay-safe.png)

![chall](https://raw.githubusercontent.com/3r0th3r-CC/3r0th3r-CC.github.io/master/source/assets/images/posts/VishwaCTF-2025/Web/scan-it-to-stay-safe-chall.png)

This is a web chall that will scan for any website to check if it has spam content. But is it filter internal addresses like `127.0.0.1`?

![localhost](https://raw.githubusercontent.com/3r0th3r-CC/3r0th3r-CC.github.io/master/source/assets/images/posts/VishwaCTF-2025/Web/scan-it-to-stay-safe-127.0.0.1.png)

It seems that it didn't block the internal address, however an error exists. As I'm familiar with this vuln, This error can be the port 80 is not the port used to host, that why it can't connect

So I started with the scan port process. The main point of this scan is to find out which port the server use to host the web. Or if we're lucky enough, we can found some port from Redis, FastCGI, Zabbix,...(@_@)

You can write a Python script to scan or if you have Burp Suite Pro, follow these:

Enter http://127.0.0.1:1 or any other port and use Burp Suite to intercept the request. Here you can see a hint "Follow the trail..." in the cookie. However, I didn't know what it means so just skip it =))

![hint](https://raw.githubusercontent.com/3r0th3r-CC/3r0th3r-CC.github.io/master/source/assets/images/posts/VishwaCTF-2025/Web/scan-it-to-stay-safe-hint.png)

Next, press **Ctrl + I** to send it to **Intruder**

Then, add the position, where the port will be inserted

![position](https://raw.githubusercontent.com/3r0th3r-CC/3r0th3r-CC.github.io/master/source/assets/images/posts/VishwaCTF-2025/Web/scan-it-to-stay-safe-position.png)

After that, in the Payloads tab, choose **Payloads type** is **Numbers** with range from `0` to `65536`

Finally, click **Start attack** and wait to see if there is any difference in response length

After scanning, I found `port 5000`, yay

![port5000](https://raw.githubusercontent.com/3r0th3r-CC/3r0th3r-CC.github.io/master/source/assets/images/posts/VishwaCTF-2025/Web/scan-it-to-stay-safe-port-5000.png)

The next idea is to found some hidden paths in `port 5000`, and here I use dirb wordlist

Later, I found `/console`. However, after some tests with the internal **Werkzeug**, I wasn't able to execute commands or exploit path traversal to get the flag :(

Back to the hint, I thought I must have missed something. So, I tested again, this time using **webhook.site** to receive the response. I guessed that **follow the trail** likely meant to follow the response

And yea, my guess was right, the flag was on the response header, I just overlooked everything :(

![flag](https://raw.githubusercontent.com/3r0th3r-CC/3r0th3r-CC.github.io/master/source/assets/images/posts/VishwaCTF-2025/Web/scan-it-to-stay-safe-flag.png)

> **FLAG: VishwaCTF{Y0u_7R4c30lI7_3000_4rK}**

> Sorry organizer for aggressive scan O_O

## Are We Up?

![AreWeUp](https://raw.githubusercontent.com/3r0th3r-CC/3r0th3r-CC.github.io/master/source/assets/images/posts/VishwaCTF-2025/Web/are-we-up.png)

At first, I had no idea how to proceed since there was no instance to examine and no clues to follow, so I skipped this chall

But later, a hint release. It told me that I must look at the status of **vishwactf.com**

Hmm, what does "status" mean? I don't know so I just randomly try some path like `/health-check`, `/health`,... until I saw a green dot at the end, it right on the main page

![status](https://raw.githubusercontent.com/3r0th3r-CC/3r0th3r-CC.github.io/master/source/assets/images/posts/VishwaCTF-2025/Web/are-we-up-status.png)

This actually lead me to a website **web.eng.run:8501** or **are-we-up-3dd2awd2.ctf-delivery-net.pp.ua** (after the contest end)

On this page, I found a `/flag` endpoint and a `local:8080`, but it turned out to be a fake flag :(

![fake_flag](https://raw.githubusercontent.com/3r0th3r-CC/3r0th3r-CC.github.io/master/source/assets/images/posts/VishwaCTF-2025/Web/are-we-up-fake-flag.png)

~~VishwaCTF{L0L_7H15_15_N07_7H3_Fl49}~~

Look at the source code, I found `/uptimer-a343s19` endpoint

![Uptimer](https://raw.githubusercontent.com/3r0th3r-CC/3r0th3r-CC.github.io/master/source/assets/images/posts/VishwaCTF-2025/Web/are-we-up-uptimer.png)

Alright, look like we have another SSRF challenge ~

My first try is http://127.0.0.1:8080/flag but no luck, it forbidden

After some testing, I realized some filters like only allow http, https or block `127.0.0.1`,...

At first, I focused on the ip address and try to turn it to `0.0.0.0`, `2130706433`,... but none of those worked

I started brainstorming and realized I had overlooked something: what about redirects? I attempted to set up a host that could redirect, but since I'm broke, you know...

So I tried using ngrok, but somehow it didn't work

Then I tried another easier method, other people host :)

As you can see in the [PayloadAllTheThings](https://github.com/swisskyrepo/PayloadsAllTheThings/tree/master/Server%20Side%20Request%20Forgery#bypassing-using-a-redirect), there is an url https://307.r3dir.me/--to/?url=http://localhost, which can redirect

When I tried that, another different error occur

![error](https://raw.githubusercontent.com/3r0th3r-CC/3r0th3r-CC.github.io/master/source/assets/images/posts/VishwaCTF-2025/Web/are-we-up-error.png)

Then, I changed to https://307.r3dir.me/--to/?url=http://localhost:8080/flag

However, this time it return the fake flag. This made me wonder if the internal server also had filters in place. So I changed to https://307.r3dir.me/--to/?url=http://127.0.0.1:8080/flag and I got the real flag

> **FLAG: VishwaCTF{Y0r4_lo7al_b4bby_A4k18aw2}**

## forgot-h1-login

![forgot-h1-login](https://raw.githubusercontent.com/3r0th3r-CC/3r0th3r-CC.github.io/master/source/assets/images/posts/VishwaCTF-2025/Web/forgot-h1-login.png)

Like every other web chall, I recon the web, and I realized that I must focus on the login page because it was the only available feature here :)

![forgot-h1-login](https://raw.githubusercontent.com/3r0th3r-CC/3r0th3r-CC.github.io/master/source/assets/images/posts/VishwaCTF-2025/Web/forgot-h1-login-login.png)

Look at the source of `/login`, I saw `/dashboard` and `/auth` path

![forgot-h1-login-path](https://raw.githubusercontent.com/3r0th3r-CC/3r0th3r-CC.github.io/master/source/assets/images/posts/VishwaCTF-2025/Web/forgot-h1-login-path.png)

In `/dashboard`, I look at the source code again and I found an email in the comment: `ark.dev@hackerone.com`

Back to the login page, there is another feature, **Reset Password**. It require to enter the email, and of course, I will try the `ark.dev@hackerone.com` first no matter what

![otp](https://raw.githubusercontent.com/3r0th3r-CC/3r0th3r-CC.github.io/master/source/assets/images/posts/VishwaCTF-2025/Web/forgot-h1-login-otp.png)

Here, it returns an OTP form. However, brute force a long OTP code seem impossible, so I must find some other way to get the code

I use **Burp Suite** to intercept the reset password request first to investigate more

![intercept](https://raw.githubusercontent.com/3r0th3r-CC/3r0th3r-CC.github.io/master/source/assets/images/posts/VishwaCTF-2025/Web/forgot-h1-login-intercept.png)

Hmm, wait a minute, I noticed some thing similar, it was actually same as my previous writeup here: https://t3l3sc0p3.github.io/posts/knightctf-2024-writeup/#gain-access-2-440-pts

The idea is actually easy, you just need to add your email next to the valid one, then add `[]` to turn it to an array. This way, the code will be return to both of them

![bypass](https://raw.githubusercontent.com/3r0th3r-CC/3r0th3r-CC.github.io/master/source/assets/images/posts/VishwaCTF-2025/Web/forgot-h1-login-bypass.png)

With experience from **scan-it-to-stay-safe** challenge, I use webhook email instead of my ctf email just to make sure I'm not missing any thing hehe

And with this caution, I got the flag very fast. If I remember correctly, I was the **6-7th** solver

![nice](https://i.imgur.com/jwYlN9G.gif)

![flag](https://raw.githubusercontent.com/3r0th3r-CC/3r0th3r-CC.github.io/master/source/assets/images/posts/VishwaCTF-2025/Web/forgot-h1-login-flag.png)

> **FLAG: VishwaCTF{y0u_4r3_7h3_h4ck3r_numb3r_0n3_2688658}**

# Digital Forensics

## Leaky Stream

![LeakyStream](https://raw.githubusercontent.com/3r0th3r-CC/3r0th3r-CC.github.io/master/source/assets/images/posts/VishwaCTF-2025/Forensics/leaky-stream.png)

I'm a web player, not a forensics player, so when I saw these type of challenge, first I will `nano chitty-chat.pcapng` on terminal then search for `VishwaCTF{`

![firstpart](https://raw.githubusercontent.com/3r0th3r-CC/3r0th3r-CC.github.io/master/source/assets/images/posts/VishwaCTF-2025/Forensics/leaky-stream-first-part.png)

I found: `VishwaCTF{this_is_first_part`

Then, I thought there might be a second part of this somewhere, so I search for "second" and I got the flag, sound dumb but this is actually how I solve this chall (o_O)

![secondpart](https://raw.githubusercontent.com/3r0th3r-CC/3r0th3r-CC.github.io/master/source/assets/images/posts/VishwaCTF-2025/Forensics/leaky-stream-second-part.png)

> **FLAG: VishwaCTF{this_is_first_part_this_second_part}**

## Persist

![Persist](https://raw.githubusercontent.com/3r0th3r-CC/3r0th3r-CC.github.io/master/source/assets/images/posts/VishwaCTF-2025/Forensics/persist.png)

I opened the **HKCU** and **SOFTWARE** hives with **Registry Explorer**. Then, I pressed **Ctrl + F** and searched for suspicious keywords—specifically, `powershell`

When I scrolled to `Microsoft\Windows\CurrentVersion\Run`, it caught my attention

![attention](https://raw.githubusercontent.com/3r0th3r-CC/3r0th3r-CC.github.io/master/source/assets/images/posts/VishwaCTF-2025/Forensics/persist-attention.png)

It runs a command `C:\Windows\System32\WindowsPowerShell\v1.0\powershell.exe -enc` with a **Base64-encoded** string, which looks very suspicious

![base64](https://raw.githubusercontent.com/3r0th3r-CC/3r0th3r-CC.github.io/master/source/assets/images/posts/VishwaCTF-2025/Forensics/persist-base64.png)

After decoding it, I found that it was a **Vigenère cipher** decryption function

![cipher](https://raw.githubusercontent.com/3r0th3r-CC/3r0th3r-CC.github.io/master/source/assets/images/posts/VishwaCTF-2025/Forensics/persist-vigere-cipher.png)

I tried searching for something like a cipher text and a key but found nothing. However, after looking more carefully, I finally found what I needed in: `HKCU:Software\Microsoft\Windows\CurrentVersion\Explorer\RecentDocs`

![flag](https://raw.githubusercontent.com/3r0th3r-CC/3r0th3r-CC.github.io/master/source/assets/images/posts/VishwaCTF-2025/Forensics/persist-flag.png)

> **FLAG: VishwaCTF{b3l1ef_in_r3g_p0wer}**

## Whispers

After the analysis saw the **ftp** packets and the file transmission, I went to look at it

![Whispers](https://raw.githubusercontent.com/3r0th3r-CC/3r0th3r-CC.github.io/master/source/assets/images/posts/VishwaCTF-2025/Forensics/whispers.png)

After that, I tried to crack but did not work so I tried to read the **TCP packets** and saw it​

![tcp](https://raw.githubusercontent.com/3r0th3r-CC/3r0th3r-CC.github.io/master/source/assets/images/posts/VishwaCTF-2025/Forensics/whispers-tcp.png)

Look carefully, you will see the zip's password is the first letter of each line​

After unzip I saw the files but when I opened `hehehe.jpg` file, I couldn't

So I check what code it is written by

![file](https://raw.githubusercontent.com/3r0th3r-CC/3r0th3r-CC.github.io/master/source/assets/images/posts/VishwaCTF-2025/Forensics/whispers-file.png)

![diec](https://raw.githubusercontent.com/3r0th3r-CC/3r0th3r-CC.github.io/master/source/assets/images/posts/VishwaCTF-2025/Forensics/whispers-diec.png)

Now, use `pyinstxtractor` to extract and you will got the flag:

![pyinstxtractor](https://raw.githubusercontent.com/3r0th3r-CC/3r0th3r-CC.github.io/master/source/assets/images/posts/VishwaCTF-2025/Forensics/whispers-pyinstxtractor.png)

![flag](https://raw.githubusercontent.com/3r0th3r-CC/3r0th3r-CC.github.io/master/source/assets/images/posts/VishwaCTF-2025/Forensics/whispers-flag.png)

> **FLAG: VishwaCTF{h1dd3n_l0c4l_traff1c}**

# Reverse Engineering

## Hungry Friends

![hungry-friends](https://raw.githubusercontent.com/3r0th3r-CC/3r0th3r-CC.github.io/master/source/assets/images/posts/VishwaCTF-2025/Reverse/hungry-friends.png)

This is a challenge where, when I run the file, I encounter a bunch of errors

![error](https://raw.githubusercontent.com/3r0th3r-CC/3r0th3r-CC.github.io/master/source/assets/images/posts/VishwaCTF-2025/Reverse/hungry-friends-error.png)

After going to https://www.dllme.com to download all of the necessary DLL files, I was finally able to run the program

![game](https://raw.githubusercontent.com/3r0th3r-CC/3r0th3r-CC.github.io/master/source/assets/images/posts/VishwaCTF-2025/Reverse/hungry-friends-game.png)

This is a simple Snake game. After throwing it into **IDA** for analysis, I found that at address `0x00401C1F`, it checks whether you have at least **9999** points. If the condition is met, it calls the `SHOW_FLAG` function

![9999](https://raw.githubusercontent.com/3r0th3r-CC/3r0th3r-CC.github.io/master/source/assets/images/posts/VishwaCTF-2025/Reverse/hungry-friends-9999.png)

Now, the remaining task is to set a breakpoint at `0x00401C1F` and debug it in **IDA**, then set `EAX = 9999` and continue

![modify](https://raw.githubusercontent.com/3r0th3r-CC/3r0th3r-CC.github.io/master/source/assets/images/posts/VishwaCTF-2025/Reverse/hungry-friends-modify.png)

![flag](https://raw.githubusercontent.com/3r0th3r-CC/3r0th3r-CC.github.io/master/source/assets/images/posts/VishwaCTF-2025/Reverse/hungry-friends-flag.png)

> **FLAG: VishwaCTF{th3r3_4r3_5n4k35_all_4r0und}**

## Safe Box

![SafeBox](https://raw.githubusercontent.com/3r0th3r-CC/3r0th3r-CC.github.io/master/source/assets/images/posts/VishwaCTF-2025/Reverse/safe-box.png)

This is a reverse engineering challenge involving a **Unity** game. The goal of the challenge is to find the password to unlock a safe and retrieve the flag

Typically, I start by locating the `Assembly-CSharp.dll` file and analyzing it wit **dnSpy**

When inspecting the source code, I noticed it was obfuscated. To make it more readable, I opened it in **VSCode** and renamed variables and functions

While analyzing the code, I found features such as checking whether `Assembly-CSharp.dll` has been modified, verifying the entered password using arithmetic operations like **ADD**, **SUB**, **XOR**,... and a few other functions

I modified the password verification logic by changing `==` to `!=`, saved the changes, and ran the game, but nothing changed

After some time, I tried deleting `Assembly-CSharp.dll`, and surprisingly, the game logic still worked

This led me to investigate other DLL files, and when I checked `UnityEngine.NetworkUtils.dll`, I finally found the actual logic handling the password verification

![code](https://raw.githubusercontent.com/3r0th3r-CC/3r0th3r-CC.github.io/master/source/assets/images/posts/VishwaCTF-2025/Reverse/safe-box-code.png)

After modifying the logic, the final step is to open the game, press **OK** to unlock the safe, and retrieve the flag

![flag](https://raw.githubusercontent.com/3r0th3r-CC/3r0th3r-CC.github.io/master/source/assets/images/posts/VishwaCTF-2025/Reverse/safe-box-flag.png)

> **FLAG: VishwaCTF{h3r3_y0u_@r3}**

# Cryptography

## Aria of lost code

![Aria-of-lost-code](https://raw.githubusercontent.com/3r0th3r-CC/3r0th3r-CC.github.io/master/source/assets/images/posts/VishwaCTF-2025/Crypto/aria-of-lost-code.png)

When I opened this challenge, I initially thought it must be some kind of cipher on **dCode**. But instead of checking it on **dCode**, I decided to use Google Lens and boommm @@

I found the "Hymmnos alphabet"!!

![Google Lens](https://raw.githubusercontent.com/3r0th3r-CC/3r0th3r-CC.github.io/master/source/assets/images/posts/VishwaCTF-2025/Crypto/aria-of-lost-code-google.png)

Once I identified the cipher, I went to [dCode's Hymmnos alphabet page](https://www.dcode.fr/hymnos-alphabet), entered the ciphertext, and easily decoded the flag

![flag](https://raw.githubusercontent.com/3r0th3r-CC/3r0th3r-CC.github.io/master/source/assets/images/posts/VishwaCTF-2025/Crypto/aria-of-lost-code-flag.png)

> **FLAG: VishwaCTF{H4v3_y0u_7ri3d_Ar_70n3l1c0}**

## Chaos

![chaos](https://raw.githubusercontent.com/3r0th3r-CC/3r0th3r-CC.github.io/master/source/assets/images/posts/VishwaCTF-2025/Crypto/chaos.png)

When I first read this challenge, the first thing I saw was chaos :) (Okay, that's a good joke, right? :D)

After opening `output.txt`, I saw a bunch of random characters that maybe the flag was encrypted and hidden somewhere in there

Moving on to `challenge.py`, I noticed that the flag was shuffled randomly within the messages. Each message was encrypted using **XOR** and **Base85**

![encoded](https://raw.githubusercontent.com/3r0th3r-CC/3r0th3r-CC.github.io/master/source/assets/images/posts/VishwaCTF-2025/Crypto/chaos-xor-base85.png)

After analyzing the problem, I decided to write a script to decode the messages and find the flag:

1. Reverse **Base85** to obtain the **XOR-encrypted** bytes
2. Reverse the **XOR** operation using `(i % 256)`, which revealed the messages mixed with the flag

```python
#!/usr/bin/python3
import base64

def xor_decrypt(encoded_msg):
    # Decode from Base85 and reverse the XOR (each byte XOR'ed with its index modulo 256)
    decoded = base64.b85decode(encoded_msg)
    transformed = bytearray(decoded)
    for i in range(len(transformed)):
        transformed[i] ^= (i % 256)
    return transformed.decode(errors='ignore')

# Read the encrypted messages from the file, separated by double newlines
with open('output.txt', 'r') as f:
    encrypted_messages = f.read().strip().split('\n\n')

# Decrypt each message and print them
for i, msg in enumerate(encrypted_messages, 1):
    print(f"Message {i}: {xor_decrypt(msg)}")
```

After running the script, I obtained the messages and several possible flags :D (You heard it right, "some flags")

![flags](https://raw.githubusercontent.com/3r0th3r-CC/3r0th3r-CC.github.io/master/source/assets/images/posts/VishwaCTF-2025/Crypto/chaos-flags.png)

Some flag:

```txt
VishwaCTF{Fl4g_Or_N0t_Th4t_1s_Th3_QuesT10n}​
VishwaCTF{T00_M4ny_F4k3_Fl4Gs_G00d_Luck}
VishwaCTF{CrYpt0_cRyPT0_1g_It_1s_sOm3_7hiNg_t0_D0}
VishwaCTF{NoT_ThE_ReaL_fLaG_K33P_tRy1Ng}
VishwaCTF{Y0u_WiLl_N3v3r_F1nd_1t}
```

I started with the longest one and, luckily, I found the real flag >:3​


> **FLAG: VishwaCTF{CrYpt0_cRyPT0_1g_It_1s_sOm3_7hiNg_t0_D0}**

## Forgotten Cipher

![ForgottenCipher](https://raw.githubusercontent.com/3r0th3r-CC/3r0th3r-CC.github.io/master/source/assets/images/posts/VishwaCTF-2025/Crypto/forgotten-cipher.png)

This challenge involved the **VIC cipher** and **XOR transformations**, which I found quite fun. After researching the **VIC cipher** (actually with a little help from AI :3), I realized that I needed to:

1. Find the base key, which was not provided
2. Reverse the bitwise transformations
    a. If `i` is even, apply rotate right by 2 bits
    b. If `i` is odd, apply rotate left by 2 bits

3. Reverse the **XOR** operation with the initial key
4. Check if the result is valid **ASCII**
5. Brute-force the `base_key` from **0** to **255** to find the correct value

Based on the analysis above (and some help from ~~ChatGPT~~ AI >:3), I wrote a **Python** script to solve the problem:

```python
#!/usr/bin/python3
import string

cipher_hex = "0d4ac648a2f0bee7bccf0231c35e13ba7bc93a2d8f7d9498885e3f4998"
cipher = bytes.fromhex(cipher_hex)

def rotate_left_2(x):
    return ((x << 2) & 0xFF) | (x >> 6)

def rotate_right_2(x):
    return ((x >> 2) & 0xFF) | ((x & 0x03) << 6)

def find_flag():
    for base_key in range(256):  # Brute-force from 0x00 to 0xFF
        K = base_key
        out = []
        for i, c in enumerate(cipher):
            # Compute K(i)
            if i == 0:
                Ki = K
            else:
                Ki = (K * 3 + i) % 256
            K = Ki  # Update K for the next iteration

            # Reverse bitwise shift
            if i % 2 == 0:
                temp = rotate_right_2(c)  # Even index → rotate right 2
            else:
                temp = rotate_left_2(c)   # Odd index → rotate left 2

            # XOR to retrieve the original byte
            p = temp ^ Ki
            out.append(p)

        plaintext = bytes(out)

        # Check if the result is valid (ASCII + flag format)
        if b'VishwaCTF{' in plaintext:
            print(f"FOUND! Base key: {hex(base_key)} -> {plaintext.decode()}")
            return  # Stop when the flag is found

find_flag()
```

After running the script, I got the result:

```txt
FOUND! Base key: 0x15 -> VishwaCTF{VIC_Decoded_113510}​
```

> **FLAG: VishwaCTF{VIC_Decoded_113510}**

## Rhythmic Cipher

![rhythmic-cipher](https://raw.githubusercontent.com/3r0th3r-CC/3r0th3r-CC.github.io/master/source/assets/images/posts/VishwaCTF-2025/Crypto/rhythmic-cipher.png)

When I came to this challenge, the first thing I saw was two **GIFs** of dancing stickmen

At first, I thought, "Maybe overlaying these two might reveal the text in the correct flag order" Hah~ too inexperienced

After suffering with the first approach, I searched for **dancing stickman cipher** and found this:

![dancing-cipher](https://raw.githubusercontent.com/3r0th3r-CC/3r0th3r-CC.github.io/master/source/assets/images/posts/VishwaCTF-2025/Crypto/rhythmic-cipher-dancing-cipher.png)

Good job, myself! Gud gud :). After detecting the cipher, I thought about dumping these **GIFs** into images

Using the first tool I found on the internet to convert **GIFs** into images, I got these:

![gif1](https://raw.githubusercontent.com/3r0th3r-CC/3r0th3r-CC.github.io/master/source/assets/images/posts/VishwaCTF-2025/Crypto/rhythmic-cipher-gif1.png)

![gif2](https://raw.githubusercontent.com/3r0th3r-CC/3r0th3r-CC.github.io/master/source/assets/images/posts/VishwaCTF-2025/Crypto/rhythmic-cipher-gif2.png)

Then I decoded these with [Dcode's Dancing Men Cipher](https://www.dcode.fr/dancing-men-cipher)

![flag](https://raw.githubusercontent.com/3r0th3r-CC/3r0th3r-CC.github.io/master/source/assets/images/posts/VishwaCTF-2025/Crypto/rhythmic-cipher-flag.png)

> **FLAG: VishwaCTF{CIPHERED_DANCE}**

# OSINT

## Follow for Clues

![follownotfollow](https://raw.githubusercontent.com/3r0th3r-CC/3r0th3r-CC.github.io/master/source/assets/images/posts/VishwaCTF-2025/OSINT/follow-the-clues.png)

So at first, I will find **LinkedIn**, **Instagram** and **Twitter** of **VishwaCTF**, of course ._.

![found](https://raw.githubusercontent.com/3r0th3r-CC/3r0th3r-CC.github.io/master/source/assets/images/posts/VishwaCTF-2025/OSINT/follow-the-clues-social-media.png)

I can find all of it through the main page vishwactf.com except **Twitter (X)**
- **LinkedIn**: https://www.linkedin.com/company/cybercell-viit/
- **Instagram**: https://www.instagram.com/cybercell_viit/

About **X (Twitter)**, I just need to search on Google "VishwaCTF Twitter" and I got the link
- **Twitter (X)**: https://x.com/cybercellviit

However, I haven't found any flags in posts on **LinkedIn** or **Twitter (X)**. The only remaining social media platform to check is **Instagram**

My strategy for finding the flags is straightforward: I look at the newest posts and then way back to the oldest ones, starting from when the contest began

Then I found this post: https://www.instagram.com/p/DGXe34GvDjq/

![flag](https://raw.githubusercontent.com/3r0th3r-CC/3r0th3r-CC.github.io/master/source/assets/images/posts/VishwaCTF-2025/OSINT/follow-the-clues-flag.png)

> **FLAG: VishwaCTF{L3t_Th3_hUn7_8Eg1n}**

## Stadium!!

![Stadium](https://raw.githubusercontent.com/3r0th3r-CC/3r0th3r-CC.github.io/master/source/assets/images/posts/VishwaCTF-2025/OSINT/stadium.png)

![Challenge](https://raw.githubusercontent.com/3r0th3r-CC/3r0th3r-CC.github.io/master/source/assets/images/posts/VishwaCTF-2025/OSINT/stadium-chall.png)

I started to search it using Google Images and I found a bunch of posts about this place

![google](https://raw.githubusercontent.com/3r0th3r-CC/3r0th3r-CC.github.io/master/source/assets/images/posts/VishwaCTF-2025/OSINT/stadium-google.png)

To verify the exact name, I use Google Maps and search "Sailing Cricket Ground" and I got the full name is `Saling Cricket Stadium Ghanche`

![location](https://raw.githubusercontent.com/3r0th3r-CC/3r0th3r-CC.github.io/master/source/assets/images/posts/VishwaCTF-2025/OSINT/stadium-located.png)

> **FLAG: VishwaCTF{Saling_Cricket_Stadium_Ghanche}**

## The Lecture Code

![thelecturecode](https://raw.githubusercontent.com/3r0th3r-CC/3r0th3r-CC.github.io/master/source/assets/images/posts/VishwaCTF-2025/OSINT/the-lecture-code.png)

Look carefully on the description, I noticed something different than other challs

First, the author only found on this chall. Second, he/she also include `@riya_shah28`, which want to tell us "Hey, this is my username, you have to find me"

So I will find as your request :)

At https://vishwactf.com/heroes, I found **Riya**

![Riya](https://raw.githubusercontent.com/3r0th3r-CC/3r0th3r-CC.github.io/master/source/assets/images/posts/VishwaCTF-2025/OSINT/the-lecture-code-riya.png)

Look at her **LinkedIn** and look to the comment section, I saw the "cryptic message":

![cryptic](https://raw.githubusercontent.com/3r0th3r-CC/3r0th3r-CC.github.io/master/source/assets/images/posts/VishwaCTF-2025/OSINT/the-lecture-code-cryptic-message.png)

Then, I look at her **GitHub**, I found a **cs50cybersecurity** repo: https://github.com/ri285/cs50cybersecurity

Here, I found 3 **Python** file but only `crack2.py` actually works

So I clone it to my machine, replace the `hidden_encrypted_pass` and then `python crack2.py` :3

![flag](https://raw.githubusercontent.com/3r0th3r-CC/3r0th3r-CC.github.io/master/source/assets/images/posts/VishwaCTF-2025/OSINT/the-lecture-code-flag.png)

> **FLAG: VishwaCTF{cs5o_qu4ck!}**

## The Summit

![thesummit](https://raw.githubusercontent.com/3r0th3r-CC/3r0th3r-CC.github.io/master/source/assets/images/posts/VishwaCTF-2025/OSINT/the-summit.png)

![Challenge](https://raw.githubusercontent.com/3r0th3r-CC/3r0th3r-CC.github.io/master/source/assets/images/posts/VishwaCTF-2025/OSINT/the-summit-chall.png)

So from the information of the image, we got something like "2025 01 05 11:57" and **TheMIS**. Search on Google, I found that this might be a type of tank or at least, use in military

![tank](https://raw.githubusercontent.com/3r0th3r-CC/3r0th3r-CC.github.io/master/source/assets/images/posts/VishwaCTF-2025/OSINT/the-summit-tank.png)

Look at the left, I saw at lot of people and from my experience when playing **GeoGuessr**, this is **Indians**

![indians](https://raw.githubusercontent.com/3r0th3r-CC/3r0th3r-CC.github.io/master/source/assets/images/posts/VishwaCTF-2025/OSINT/the-summit-indians.png)

So from all of these information and the description, I think that we have to find an important event in India?

Initially, our team was in vain because it too vague. But then later, a member in team noticed this in the description: "with a lot of attention from the media"

This means that it might be some where on the social media. And with a little information is "2025 01 05", in the last 2 hours, my team search all the social media from **Instagram**, **Twitter**, **LinkedIn**, **Facebook** to find the event in most Indian's military, army page

Until we got this post: https://www.facebook.com/CMOMaharashtra/posts/pfbid0ZSfoYhYaeUbSJVimCj7DcMhtZMKKryfMKnM9ZX3F5QmV9LiRxrrpcktzK2Ys4VhAl

![post](https://raw.githubusercontent.com/3r0th3r-CC/3r0th3r-CC.github.io/master/source/assets/images/posts/VishwaCTF-2025/OSINT/the-summit-post.png)

This is "Know Your Army" Mela event, and in the original post, it got a lot of attention: https://www.facebook.com/Indianarmy.adgpi/posts/pfbid02c5tc9pYDibfGCeYtNuz6FaEFUQotKjZJx3dgaojYvejJoQNts83nqTX2ECEjvSHql

Here, it also has Google Maps, which located the exact long lat of the location: https://maps.app.goo.gl/KMjNrG8FQMEp2n8z8

![longlat](https://raw.githubusercontent.com/3r0th3r-CC/3r0th3r-CC.github.io/master/source/assets/images/posts/VishwaCTF-2025/OSINT/the-summit-longlat.png)

Look like we are on the right track, how about the guest?

As there is only a little time, so a member of our team use ~~ChatGPT~~ AI to fast search and found:

![gpt](https://raw.githubusercontent.com/3r0th3r-CC/3r0th3r-CC.github.io/master/source/assets/images/posts/VishwaCTF-2025/OSINT/the-summit-gpt.png)

We finally got the flag uwu

> **FLAG: VishwaCTF{18.51,73.89_Devendra Fadnavis}**

# Steganography

## Lets Race

![lets racing](https://raw.githubusercontent.com/3r0th3r-CC/3r0th3r-CC.github.io/master/source/assets/images/posts/VishwaCTF-2025/Steg/lets-race.png)

First, I cracked password with `join`

![crack](https://raw.githubusercontent.com/3r0th3r-CC/3r0th3r-CC.github.io/master/source/assets/images/posts/VishwaCTF-2025/Steg/lets-race-crack.png)

Based on this description, there was a lot of time later that I tried based on the current rankings of the household `(R1 G2 B3)`

![rgb](https://raw.githubusercontent.com/3r0th3r-CC/3r0th3r-CC.github.io/master/source/assets/images/posts/VishwaCTF-2025/Steg/lets-race-rgb.png)

![car](https://raw.githubusercontent.com/3r0th3r-CC/3r0th3r-CC.github.io/master/source/assets/images/posts/VishwaCTF-2025/Steg/lets-race-car.png)

Then, I use `stegsolve` and I got the flag:

![flag](https://raw.githubusercontent.com/3r0th3r-CC/3r0th3r-CC.github.io/master/source/assets/images/posts/VishwaCTF-2025/Steg/lets-race-flag.png)

> **FLAG: VishwaCTF{1_l0v3_C0r5}**

## Quadrant

![quadrant](https://raw.githubusercontent.com/3r0th3r-CC/3r0th3r-CC.github.io/master/source/assets/images/posts/VishwaCTF-2025/Steg/quadrant.png)

I opened the file with **GIMP** and combined four images together, then fixed a few missing parts

![qrcode](https://raw.githubusercontent.com/3r0th3r-CC/3r0th3r-CC.github.io/master/source/assets/images/posts/VishwaCTF-2025/Steg/quadrant-fixed.png)

> **FLAG: VishwaCTF{aG9lMTIzNDU2c3Bhc3NhZ2U=}**

## Spilled Paint Water

![spilled](https://raw.githubusercontent.com/3r0th3r-CC/3r0th3r-CC.github.io/master/source/assets/images/posts/VishwaCTF-2025/Steg/spilled-paint-water.png)

I opened the file on **svgviewer.dev** and then deleted a few lines like the ones below

```html
<clipPath id="clipArea">
    <rect x="1000" y="1000" width="200" height="200"/>
</clipPath>
```

and

```css
svg {
    background-color:#a2b5e0 ;
}
```

![flag](https://raw.githubusercontent.com/3r0th3r-CC/3r0th3r-CC.github.io/master/source/assets/images/posts/VishwaCTF-2025/Steg/spilled-paint-water-fixed.png)

> **FLAG: VishwaCTF{STROKE__N_FILL}**

![i need some head pats](https://i.imgur.com/fGWriq1.jpeg)

Thanks for reading guys, really appreciate, it was pretty long right?
