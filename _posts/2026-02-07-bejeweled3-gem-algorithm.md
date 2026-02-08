---
layout: post
title: How does Bejeweled 3 determine the next set of gems to drop?
description: "Reverse engineering Bejeweled 3 to find out how Zen mode guarantees you can always keep playing — turns out it's a 112-operation brute force check."
image: /assets/20260207_235739_image.png
---

![](/assets/20260207_235739_image.png)

Bejeweled 3 is a match-three puzzle game by PopCap Games.  The game has several modes -- Classic gives you a time limit and you can lose by board deadlocks, while Zen lets you play indefinitely with no lose condition.

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

The game does not use a single Board class with a mode flag. Each mode is a separate subclass, and mode-specific behaviour lives in virtual method overrides.

## Tracking down the function

Searching `Seed : %d` leads to a function that initializes a large object, sets up an 8x8 grid at an offset, and seeds an RNG. It's a Mersenne Twister implementation (624 is a special number in MT19937):

![](/assets/20260207_204703_image.png)

![](/assets/20260207_204806_image.png)

![](/assets/20260207_201245_image.png)

Tracking down the Board::Board constructor, we can find four calls to this function, one of which contains Zen.

![](/assets/20260207_201602_image.png)

I diffed the vtables between ClassicBoard and ZenBoard to find the gem assignment method, but the overridden methods had nothing to do with gem generation -- they handled tick updates, binaural beats, mantra text, and audio effects. Very Zen.

That meant the gem drop algorithm had to be in the base Board class, with its behaviour controlled by a parameter.

I went back up to the RNG function and started to check where it was xref'd, one of which took Board*, and a special parameter a2...

The decompiled output is ~500 lines. Most of it is irrelevant. The RNG xref puts the cursor right at the interesting part:

![](/assets/20260207_205004_image.png)
*Suspicious large loop*

 Reading outward from the RNG call, there's a subfunction with direction offsets (-1, 0, 1), bounds checks against an 8x8 grid, and match-length comparisons:

![](/assets/20260207_211652_image.png)
*v43 filled with -1s, 0s and 1s.*

![](/assets/20260207_211823_image.png)
*Bounds check? 8x8 grid*

![](/assets/20260207_212237_image.png)
*Some >=2s and >=3s?*

It's checking every possible swap. In an 8x8 grid, that's 6x6 gems with 4 way swaps, 24 edges with 3 swaps and 4 corners with 2 swaps. Total of 36\*4 + 24\*3 + 4\*2 = 224. Removing duplicates, it would be 224/2 = 112 checks. (I don't think it saves checks, and it breaks on first possible match)

This 112-bruteforce algorithm is inside an if condition gated by a vtable entry that was same for both classic and zen. Looks pointless right now, but will become important when the retry happens. Now checking a2, linking debugger to app and running Classic and Zen and checking [esp+04]

![](/assets/20260207_215023_image.png)
*Classic [a2=01]*

![](/assets/20260207_215209_image.png)
*Zen [a2=00]*

Inside the function, `a2` gates one specific check: a scan of the board for existing 3-in-a-row matches. When `a2` is 0, the function rejects any random fill that already contains a match and retries with new colors. When `a2` is 1, that check is skipped -- the first random fill is accepted and the function moves along. This is done by Classic, just take the first assignment and push it. Allows for cascades, blasts and chains to happen in normal gameplay.

**Zen** (`a2 = 0`): retries until no pre-existing 3-in-a-row is found. On each retry, the 112-swap bruteforce restores the validation flag, which also ensures at least one valid move exists. With 7 colors on 64 cells, a passing fill usually takes a handful of attempts.

The answer is bruteforce check. Now I can peacefully go back to playing Zen.

![](/assets/20260207_235757_image.png)
