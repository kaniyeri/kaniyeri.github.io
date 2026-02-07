---
layout: post
title: How does Bejeweled 3 determine the next set of gems to drop?
---

I was curious about how the Classic mode can have unsolvable grids, and end the game, whereas Zen mode is practically endless and you can never lose. I wanted to know how the game determines the next set of gems dropped in Zen mode so that you can keep playing forever.

---

Spoiler - Brute force. No fancy algorithm as I envisioned. Verification computation is only 112 operations per guess on an 8x8 grid.

---

## Reversing

Bejeweled3.exe is a simple Win32 executable with no packing and obfuscation. Extracting readable strings and looking through them gives us some candidates:

```
"Seed: %d"
"BadMovesAllowed"
"ColorCount"
"NeverAllowCascades"
"SOUND_VOICE_NOMOREMOVES"
```

The binary has RTTI (Runtime Type Information) which makes it easy to scrub through and find anchors. We can see class names in the `.data` section:

![](/assets/20260207_200442_image.png)

The `@Sexy` comes from the PopCap Games framework:

> *PopCap Games Framework* (official name *SexyApp Framework*) is a video game development kit for C++, released by PopCap Games. It is designed to let programmers easily and quickly create "PopCap-style" games, and is part of their developer program that encourages game creators to distribute their finished games through PopCap Games.
>
> — [Wikipedia, *PopCap Games*](https://en.wikipedia.org/wiki/PopCap_Games)

The game does not use a single Board class, with modes. Instead, each mode is a separate subclass, and mode-specific behaviour is in virtual method overrides.

I tried to diff the vtables between ClassicBoard and ZenBoard, to find the assignment method.

## Tracking down the function

Searching `Seed : %d` leads to a function, that initializes a huge byte object, sets up an 8x8 grid at an offset, and seeds an RNG. It's a Mersenne Twister implementation. (624 is a special number you see in MT19937):

![](/assets/20260207_204703_image.png)

![](/assets/20260207_204806_image.png)

![](/assets/20260207_201245_image.png)

Tracking down the Board::Board constructor, we can find four calls to this function, one of which contains Zen.

![](/assets/20260207_201602_image.png)

Checking each of the overridden methods, none of them did anything related to the gem generation. The overridden methods handled tick updates, binaural beats, mantra text, and audio effects. Very Zen.

That could only mean the gem drop algorithm was basing behaviour off of a parameter set by this function.

I went back up to the RNG function and started to check where it was xref'd, one of which took Board*, and a special parameter a2...

The decompiled output is ~500 lines. Most of it is irrelevant. The RNG xref puts the cursor right at the interesting part:

![](/assets/20260207_205004_image.png)
*Suspicious big loop*

A gem object's color is at offset 544. The color is picked from an array using a random index. Reading outward from here, a subfunction is called:

![](/assets/20260207_211652_image.png)
*v43 filled with -1s, 0s and 1s.*

![](/assets/20260207_211823_image.png)
*Bounds check? 8x8 grid*

![](/assets/20260207_212237_image.png)
*Some >=2s and >=3s?*

It's checking every possible swap. In an 8x8 grid, that's 6x6 gems with 4 way swaps, 24 edges with 3 swaps and 4 corners with 2 swaps. Total of 36\*4 + 24\*3 + 4\*2 = 224. Removing duplicates, it would be 224/2 = 112 checks. (I don't think it saves checks, and it breaks on first possible match)

This 112-bruteforce algorithm is inside an if condition gated by a vtable entry that was same for both classic and zen. Looks pointless right now, but will become important when the retry happens. Now checking a2, linking debugger to app and running classic and zen and checking [esp+04]

![](/assets/20260207_215023_image.png)
*Classic [a2=01]*

![](/assets/20260207_215209_image.png)
*Zen [a2=00]*

a2 varies according to the mode. If the mode is set to Zen, it runs a function where it rejects if the current fill has any 3-in-a-row already for Zen reasons. If it skipped, now the 112-bruteforce check comes into play. Classic skips all these checks, allows cascade and any random fill. Since it only runs once, the 112-bruteforce run has no impact on the final fill.

Now I can peacefully go back to playing Zen.

![](/assets/20260207_235739_image.png)

![](/assets/20260207_235757_image.png)
