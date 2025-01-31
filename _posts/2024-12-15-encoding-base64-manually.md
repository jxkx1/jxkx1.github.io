---
layout: post
title: "Manually Encoding Base64: An Academic Mystery"
subtitle: "Seriously, why is this on an exam?"
description: "Seriously, why is this on an exam?"
author: "jxkxl"
header-style: text
tags:
  - Base64
  - Tutorial
  - College
---

Let’s talk about **Base64 encoding**. Normally, this is the kind of thing you’d let a computer handle—you know, like literally every other human on the planet. But no, if you’re taking a college course on malware analysis (like me), you might find yourself in a Kafkaesque scenario where you’re asked to encode Base64 manually. Why? That’s the million-dollar question—and apparently one your professor isn’t eager to answer. Nevertheless, here we are, turning a mundane encoding process into a multi-step odyssey. Let’s dive in.

---

## Step 1: Convert Text to ASCII (Here’s the Table You’ll Need)
We’re going to encode the text **`"hi"`** into Base64. Why "hi"? Probably because even your professor realized no one has the patience for anything longer.

First, translate each character in "hi" into its ASCII value. Here’s the relevant snippet from the ASCII table, which you’ll need because apparently, memorizing this is the height of academia:

| Character | ASCII (Decimal) | ASCII (Binary) |
|-----------|------------------|-----------------|
| h         | 104              | 01101000        |
| i         | 105              | 01101001        |

So:
- `h` becomes **104**
- `i` becomes **105**

If you’re wondering why you’re expected to manually encode Base64 instead of, say, running a command or using Google, you’re not alone.

---

## Step 2: Convert ASCII to 8-Bit Binary
Next, convert those ASCII values into 8-bit binary, because Base64 loves making everything a little more inconvenient.

**104** (h) = `01101000`

**105** (i) = `01101001`

So now "hi" is represented as:

`01101000 01101001`

At this point, you might start asking yourself: *What’s the point of this exercise? Isn’t binary already cool enough?* Don’t worry; the answer remains elusive.

---

## Step 3: Split Binary Into Groups of 6 Bits
Base64 operates on 6-bit chunks, so now we take our binary string and chop it up:

`011010 000110 1001`

Oh, but what’s this? The last group only has 4 bits! That’s not good enough for Base64’s picky standards, so it’s time to pad.

---

## Step 4: Add Padding (Because Nothing Can Be Easy)
To make the last group a full 6 bits, we pad it with zeros:

`011010 000110 100100`

And that’s not the end of padding. Base64 also insists that the final result be divisible by 4 characters, so we’ll be adding `=` signs at the end (just wait for it). Padding: the gift that keeps on giving.

---

## Step 5: Convert Each Group to Decimal
Now, we take our 6-bit chunks and convert them into decimal values. Yes, because this is somehow important to your "education."

`011010` = **26**

`000110` = **6**

`100100` = **36**

---

## Step 6: Match Decimals to Base64 Characters
Here’s the magical Base64 table that maps decimal values (0–63) to specific characters. If you don’t have it memorized, congratulations! You’re a normal person.

| Decimal | Base64 Character |
|---------|-------------------|
| 0       | A                 |
| 1       | B                 |
| ...     | ...               |
| 26      | a                 |
| 36      | k                 |
| 63      | /                 |

For our decimal values:
- **26** = `a`
- **6** = `G`
- **36** = `k`

So far, our encoded result is: **`aGk`**.

---

## Step 7: Final Padding (Just When You Thought You Were Done)
Remember that padding Base64 loves so much? The final encoded string must have a length divisible by 4. If it doesn’t, we add `=` characters to make up the difference. So, we tack on one `=` to the end:

**`aGk=`**

And voilà! You’ve manually encoded "hi" in Base64. Take a moment to let the sheer uselessness of this achievement sink in.

---

## Why Are We Doing This? Seriously, Why?
Let’s face it: manually encoding Base64 is about as practical as handwriting HTML. This is a task that is—and always has been—better suited to computers. So why are we learning it? Is this some kind of hazing ritual for computer science students? An academic inside joke? A way to fill exam papers with busywork? 

Whatever the reason, the odds of you needing this skill in the real world are approximately zero. Unless your dream job is "Base64 historian," you’ll never find yourself in a situation where encoding Base64 by hand is the optimal solution. 

So the next time this pops up on an exam, just remember: you’re doing it not because it’s useful, but because someone, somewhere, thought it’d be funny to put it in the syllabus.

---

Thanks for reading! If you ever uncover the secret to why this is taught in college, please share. Inquiring minds need to know.