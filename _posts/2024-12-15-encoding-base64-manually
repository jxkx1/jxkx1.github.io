---
layout: post
title: "Manually Encoding Base64: Because Why Not"
subtitle: "A borderline unnecessary skill for your malware analysis exam"
author: "jxkxl"
header-style: text
tags:
  - Base64
  - College
  - Tutorial
---
As a web developer (or an unfortunate student taking a malware analysis exam), you may have come across **Base64 encoding**. For the uninitiated, Base64 is a binary representation scheme that lets you take regular human-readable text and turn it into a bunch of gibberish characters that look vaguely like a password you might come up with after three failed attempts.

Now, in the real world, you'd just use a programming method or an online tool to do this. But no. Why should we have things easy? Apparently, manually encoding Base64 is a “crucial skill” according to my lecturer—despite no one being able to articulate *why* this would ever matter. Let’s dive into the most extra way possible to encode Base64 manually.

---

## Step 1: Text to ASCII (Let’s Overcomplicate “hi”)
The text we’re encoding today is the very intellectual and meaningful `"hi"`. Why? Because it’s short and I refuse to make this more painful than it needs to be.

Each character in `"hi"` is converted to its ASCII value:  
`h` → **104**  
`i` → **105**

---

## Step 2: ASCII to 8-Bit Binary (Because Hexadecimal Was Too Easy)
Next, we take those ASCII values and convert them into 8-bit binary because Base64 apparently *really* enjoys making us feel like we’re living in the 80s.

**104** → `01101000`  
**105** → `01101001`

So now our text looks like this:  
`01101000 01101001`

---

## Step 3: Split into Groups of 6 Bits (Not 8, Because Why Be Logical?)
This is where things get fun (read: annoying). Base64 is all about 6-bit groups, so we chop our binary string into chunks of 6 bits:

`011010 000110 1001`

Oh wait! That last group isn’t 6 bits. Which brings us to…

---

## Step 4: Padding (Adding Zeros Like We’re Grading on a Curve)
We need to pad the final group with zeros to make it a full 6 bits:

`011010 000110 100100`

Padding doesn’t just end here, though! If we add **2 zeroes**, we slap on one `=`. If we add **4 zeroes**, we use `==`.  
(Base64 is nothing if not consistent in its inconsistency.)

---

## Step 5: Convert Each Group to Decimal (Math Makes It Better)
Time to take those 6-bit groups and convert them into decimals:

`011010` → **26**  
`000110` → **6**  
`100100` → **36**

---

## Step 6: Use the Magic Base64 Table (aka the Decoder Ring for Nerds)
To find the corresponding Base64 characters, we consult the sacred **Base64 Table** (a cryptic scroll of 64 alphanumeric characters plus `+` and `/`):

- **26** → `a`  
- **6** → `G`  
- **36** → `k`  

Thus, our binary monstrosity has been reduced to: `aGk`.

---

## Step 7: Add Padding (Because Base64 Demands a Flourish)
Remember those extra zeros we added? That means we get a single padding character: `=`.

Final result: **`aGk=`**. Congratulations! You just manually encoded `"hi"` in Base64. Don’t you feel accomplished? No? Me neither.

---

## Why Are We Doing This Again?
You may be wondering: *Why does my lecturer think this is important?* Good question. I have no idea. Maybe they’re hoping you’ll one day impress someone by manually encoding a secret message in Base64 during a cybersecurity crisis? (Spoiler alert: you won’t.) Maybe it’s just academic sadism.

Whatever the reason, now you know how to manually encode Base64. Will you ever use this skill? Unlikely. Was it fun to learn? Not really. But hey, if someone asks you to manually encode text during an exam, you’ll crush it. And if not, at least you got a sarcastic blog post out of it.

---

Thanks for reading! If you ever find a reason this skill is actually useful, please do not let me know. I do not to hear about it.
