# Maze of the Unseen - BYPASS CTF

## Challenge Information

| Field | Value |
|-------|-------|
| **CTF** | BYPASS CTF |
| **Category** | Misc |
| **Difficulty** | Medium |
| **Points** | 	250|
| **Challenge Link** | https://banana-3.onrender.com/ |

## Description

> Escape is easy.  
> Escaping the unseen isn’t.
>
> Welcome aboard a cursed maze whispered about in sailors’ tales—where invisible walls drift like ghost ships and the path you trust betrays you like a broken compass. What looks simple soon turns treacherous, luring the overconfident straight into the depths.
>
> Only those who think beyond what the eye can see—and question every step—will survive the crossing.
>
> A final note from old sea logs:  
> mosquito were not available on the ship maze
>
> Choose wisely, Captain.

**Flag format:** `BYPASS_CTF{.*}`

---

## TL;DR

The maze looks normal but contains invisible walls. Instead of following visible paths, escaping the maze requires walking *through* walls and exploring outside the intended boundaries to reach a hidden coordinate that reveals the flag.

**Vulnerability:** Invisible / fake maze boundaries  
**Approach:** Logic bypass by moving outside visible walls

---

## Initial Analysis

### Gameplay Observation

- A browser-based maze game
- Collecting checkpoints prints misleading messages like:
  `not that close baby`
- No visible exit inside the maze
- Hint suggests thinking beyond what is visible

---

## Main Analysis

### Invisible Walls Discovery

- While roaming the maze, it becomes clear:
  - Walls are not solid everywhere
  - Player can move *through* walls
- This confirms the maze is intentionally deceptive

### Exploring Outside the Maze

- Moving beyond the visible maze boundaries
- Random traversal outside the walls
- Eventually lands on a hidden coordinate

---

## Exploitation

### Strategy

1. Ignore visible maze paths
2. Walk through walls deliberately
3. Explore outside the maze area
4. Reach the secret coordinate
5. Trigger flag reveal

### Key Coordinate

- Secret coordinate reached:
  - `x = 1883`
  - `y = 404`

At this location, the game prints:

`Flag found! → BYPASS_CTF{1nv151bl3_w4ll_3sc4p3d_404}`

---

## Flag

BYPASS_CTF{1nv151bl3_w4ll_3sc4p3d_404}

