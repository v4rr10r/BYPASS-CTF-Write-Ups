#  Pirate's Treasure Hunt - BYPASS CTF 

## Challenge Information
| Field | Value |
|-------|-------|
| **CTF** | BYPASS CTF |
| **Category** | Web |
| **Difficulty** | Easy|
| **Points** | 150 |
| **Challenge Link** | http://20.196.136.66:3600/ |

## Description
>Set sail on an adventure through the treacherous seas of mathematics! Navigate through 20 nautical challenges, each more perilous than the last. Solve riddles of arithmetic under time pressure as the Kraken closes in. Only those who master the pirate code of division, multiplication, and conquest can claim the legendary pirate's map.

## Introduction

"Pirate's Treasure Hunt" is a web-based CTF challenge that immerses the player in a nautical-themed arithmetic game. The objective is to solve 20 consecutive mathematical riddles to unlock a hidden treasure (the flag). Each riddle comes with a strict time limit of 5 seconds, making manual completion extremely difficult and prone to error. The challenge is designed to test the participant's ability to analyze web traffic and automate interactions with a RESTful API.

## Challenge Analysis

Upon visiting the challenge URL, the user is presented with a "Welcome" screen. Starting the game reveals a dashboard with a progress bar, a score counter, a countdown timer, and a math question (e.g., `20 + 2 = ?`).

### Client-Side Inspection

Examining the page source and the associated `game.js` file reveals several interesting implementation details:

1.  **Anti-Debugging Measures:** The script includes functions like `disableShortcutsAndContextMenu()` and `startDevtoolsDetection()`. These attempt to prevent users from inspecting the DOM or using the browser console by detecting window resize events (often caused by opening DevTools) and blocking the UI with an overlay.
2.  **Game Logic:** The game state is managed via a `gameState` object.
3.  **API Communication:** The game communicates with the backend using `fetch` calls to specific endpoints.

### API Endpoints

By observing the network traffic (or reading the `game.js` source), we identify the following API endpoints:

*   `POST /api/game/start`:
    *   **Purpose:** Initializes a new game session.
    *   **Response:** JSON containing a `sessionId` and the first `question` object (which includes the expression and question number).

*   `POST /api/game/answer`:
    *   **Purpose:** Submits the calculated answer for the current question.
    *   **Payload:** JSON object with `sessionId` and `answer`.
    *   **Response:** JSON indicating if the answer was correct (`isCorrect`), if the game is over (`gameCompleted`), and providing either the `nextQuestion` or the `flag`.

### Vulnerability / Automation Opportunity

The "vulnerability" here is the lack of robust protection against automated requests. While the client-side code attempts to hinder inspection, the API itself is open.

The only server-side constraint observed in the client code is a minimum time threshold:
```javascript
const timeSinceQuestion = Date.now() - gameState.questionReceivedTime;
if (timeSinceQuestion < 800) {
    showFeedback('Not so fast, ye scallywag!', false);
    return;
}
```
This check prevents answers from being submitted faster than 800 milliseconds after receiving the question. However, this is easily circumvented in a script by adding a `sleep` command. Since the server relies on the client to display the 5-second countdown, a script interacting directly with the API can ensure accuracy and perfect timing without the pressure of the UI timer.

## Solution Strategy

To capture the flag, we will write a Python script to automate the gameplay. The script will perform the following steps:

1.  **Start Game:** Send a POST request to `/api/game/start` to get a session ID.
2.  **Game Loop:**
    *   Extract the arithmetic expression from the current question.
    *   Evaluate the expression (using Python's `eval()` or a custom parser).
    *   **Wait:** Sleep for at least 1 second to bypass the 800ms "anti-bot" check.
    *   **Submit:** Send the calculated answer to `/api/game/answer`.
    *   **Check Result:** If correct, proceed to the next question. If the game is completed, print the flag.

## Solution Script

The following Python script implements the strategy described above:

```python
import requests
import time

BASE_URL = "http://20.196.136.66:3600"

def solve_expression(expression):
    # Clean the string: remove " = ?" and replace 'x' with '*' if necessary
    # The challenge uses standard operators (+, -, *, /)
    expr = expression.replace(' = ?', '').replace('x', '*')
    try:
        # Evaluate the math expression
        return eval(expr)
    except Exception as e:
        print(f"Error evaluating expression: {expression} -> {e}")
        return None

def main():
    session = requests.Session()
    
    # 1. Start the game
    print("[*] Starting game...")
    try:
        response = session.post(f"{BASE_URL}/api/game/start")
        data = response.json()
    except Exception as e:
        print(f"[!] Failed to start game: {e}")
        return

    if not data.get('success'):
        print("[!] Server returned success=false")
        return

    session_id = data['sessionId']
    question = data['question']
    
    print(f"[*] Session ID: {session_id}")
    
    # 2. Loop through questions
    while True:
        print(f"[*] Question {question['number']}: {question['expression']}")
        
        # Calculate answer
        answer = solve_expression(question['expression'])
        print(f"    -> Calculated answer: {answer}")
        
        # IMPORTANT: Wait to avoid anti-bot detection (> 800ms)
        time.sleep(1.0)
        
        # Submit answer
        payload = {
            'sessionId': session_id,
            'answer': answer
        }
        
        try:
            response = session.post(f"{BASE_URL}/api/game/answer", json=payload)
            result = response.json()
        except Exception as e:
            print(f"[!] Failed to submit answer: {e}")
            return

        if not result.get('success'):
            print(f"[!] Error submitting answer: {result}")
            break
            
        if not result.get('isCorrect'):
            print("[!] Wrong answer! Game over.")
            break
            
        print("    -> Correct!")
        
        # Check for flag
        if result.get('gameCompleted'):
            print(f"\n[+] Game Completed! Flag: {result.get('flag')}")
            break
            
        # Prepare for next iteration
        question = result['nextQuestion']

if __name__ == "__main__":
    main()
```

## Execution Output

Running the script successfully navigates through all 20 islands (questions) and retrieves the flag.

```text
[*] Starting game...
[*] Session ID: d5i5h6xjmxptgo6l0hzmo9
[*] Question 1: 20 + 2
    -> Calculated answer: 22
    -> Correct!
[*] Question 2: 13 + 14
    -> Calculated answer: 27
    -> Correct!
...
[*] Question 19: 127 + (146 * 74) + 137 / 86
    -> Calculated answer: 10932.593023255815
    -> Correct!
[*] Question 20: 140 / 161 + 59 + 81 + 72
    -> Calculated answer: 212.8695652173913
    -> Correct!

[+] Game Completed! Flag: BYPASS_CTF{d1v1d3_n_c0nqu3r_l1k3_4_p1r4t3}
```

## Flag

`BYPASS_CTF{d1v1d3_n_c0nqu3r_l1k3_4_p1r4t3}`
