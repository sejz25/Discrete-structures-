# Why 23 People Are Enough: The Birthday Paradox and Hash Collisions

*A counting-and-probability topic from discrete maths that quietly shapes how long hashes need to be.*

The Saylor **CS202: Discrete Structures** course covers counting theory and probability, and its introduction says these foundations lead on to subjects like cryptology. I wanted to pick one topic where that link is visible, and the **birthday paradox** is the cleanest one I found. It's a simple probability puzzle, but it explains why a hash function can be broken far faster than you'd expect.

I didn't want to just quote the numbers, so I ran the experiments myself. Everything printed below comes from the script in this repo.

---

## The puzzle

How many people do you need in a room before there's a better-than-even chance that two of them share a birthday?

Most people guess somewhere around 183 (half of 365). The real answer is **23**.

It's called a paradox, but nothing is actually contradictory. It just feels wrong, because we think about *our own* birthday matching someone else's, when the question is about *any* two people matching.

## The maths in three lines

It's easier to calculate the opposite event, that **everyone has a different birthday**, and subtract from 1:

```text
P(all different) = (365/365) x (364/365) x (363/365) x ... x ((365 - n + 1)/365)
P(shared)        = 1 - P(all different)
```

The reason it climbs so fast is the number of **pairs**. With 23 people there are 23 x 22 / 2 = **253 pairs**, and each pair is a chance for a match. The pairs grow roughly with the square of the group size, while the group itself only grows by one at a time.

<img width="1280" height="720" alt="image" src="https://github.com/user-attachments/assets/8d2414da-0d32-4d51-bead-64417d2f6a21" />

Here's what the script printed:

```text
1) Exact probability of a shared birthday
   n = 10: 0.1169
   n = 23: 0.5073
   n = 30: 0.7063
   n = 50: 0.9704
   n = 70: 0.9992
   smallest n with probability >= 50%: 23

2) Simulation, 23 people, 100,000 rooms: 0.5040 had a shared birthday
```

The simulation (random rooms of 23) agrees with the formula, 50.4% against 50.7%. The small gap is normal random variation.

---

## From birthdays to hashes

A hash function squeezes any input into a fixed-size output. A birthday is just a hash with only 365 possible outputs. So the same logic applies: if a hash has **N** possible values, you expect your first collision after roughly

```text
sqrt(pi x N / 2)   hashes
```

which is on the order of **the square root of N**, not N itself.

That matters because a collision means two different inputs with the same fingerprint. An attacker who can find one might get a person to sign a harmless document and then swap in a malicious one with the same hash.

### My experiment

SHA-256 is far too large to collide by brute force, so I **truncated** it. I kept only the first 16, 24 and 32 bits of the output, then hashed inputs one after another until two of them matched. I repeated this many times and averaged.

```text
3) Truncated SHA-256: hashes needed before the first collision
   16-bit (65,536 possible values), 500 runs
     average hashes until collision: 319
     theory sqrt(pi*N/2):            321
     brute-force scale (N):          65,536
   24-bit (16,777,216 possible values), 500 runs
     average hashes until collision: 5,233
     theory sqrt(pi*N/2):            5,134
     brute-force scale (N):          16,777,216
   32-bit (4,294,967,296 possible values), 100 runs
     average hashes until collision: 88,373
     theory sqrt(pi*N/2):            82,137
     brute-force scale (N):          4,294,967,296
```

| Hash size | Possible values (N) | Measured average | Theory |
|---|---|---|---|
| 16-bit | 65,536 | 319 | 321 |
| 24-bit | 16,777,216 | 5,233 | 5,134 |
| 32-bit | 4,294,967,296 | 88,373 | 82,137 |

The 16-bit and 24-bit results sit very close to the formula. The 32-bit result is about 8% above it, which I put down to having only 100 runs, since each run varies a lot. I'd need more runs to tighten that up.

<img width="1280" height="720" alt="image" src="https://github.com/user-attachments/assets/280b81aa-62d8-43d1-9bc7-d4ea5b0b9f1c" />

For the 24-bit case, collisions showed up after about 5,000 hashes, even though there are almost 17 million possible values. Individual runs were all over the place, from a few hundred to nearly 14,000, but the average lands on the theory line.

---

## Why this matters in security

1. **The birthday bound halves your security level.** Finding a collision costs about the square root of the output space, so an n-bit hash gives roughly n/2 bits of collision resistance. For SHA-256 that is about 2^128 work, which is still far out of reach.
2. **Short or weak hashes get retired.** Hash functions such as MD5 and SHA-1 have had practical collision attacks demonstrated against them, and are no longer recommended where collisions matter, such as digital signatures.
3. **Digest length is chosen with this in mind.** If you want a collision-resistance level of X bits, you need a digest of about 2X bits.
4. **Hash tables and IDs.** The same maths tells you when randomly generated IDs or hash-table buckets will start colliding, which is useful well beyond cryptography.

I also want to be clear about the limits of my demo. Truncating SHA-256 to 24 bits is a classroom model. It shows the *scaling* of the birthday bound, not an attack on real SHA-256.

---

## Video

The Saylor course materials link to this explanation of the birthday paradox and the birthday attack by Steven Gordon. Clicking the thumbnail opens it on YouTube (GitHub READMEs can't embed a player).

[![Birthday paradox and birthday attack](https://img.youtube.com/vi/_JBkw60KPqw/0.jpg)](https://www.youtube.com/watch?v=_JBkw60KPqw)

<!-- TODO before posting: watch once to confirm it's the video you want to link. -->

---
## What I took away

- Counting pairs, not people, is what makes the result make sense.
- A hash that looks huge on paper is effectively half as strong against collisions.
- Running the simulation made the formula feel real. The 24-bit result landing within 2% of the theory was more convincing than any proof I could have copied.

## References

- Saylor University, *CS202: Discrete Structures*: https://learn.saylor.org/course/cs202
- Saylor University course page crediting Steven Gordon's video on the birthday paradox (CC BY 3.0): https://learn.saylor.org/mod/page/view.php?id=29598
- Matt Might, *Counting hash collisions with the birthday paradox*: https://matt.might.net/articles/counting-hash-collisions
