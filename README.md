# Recruit Madness — Expansion
A real-time, multiplayer word-pool party game where players submit words, and a rotating Messenger reads them aloud for table discussion. Built with vanilla HTML, CSS, JavaScript, and powered by Google Firebase.

---

## Table of Contents
- [How to Play](#how-to-play)
- [Core Features](#core-features)
- [Technical Architecture](#technical-architecture)
- [Setup & Installation](#setup--installation)
- [Firebase Configuration](#firebase-configuration)
- [Deployment](#deployment)
- [Key Functions Explained](#key-functions-explained)
- [System Word Pool](#system-word-pool)

---

## How to Play

### 1. Create or Join
One player creates a new room, becoming the **Host**. Other players join using the unique 4-digit Game ID. Players can reconnect to an existing session if they get disconnected.

### 2. Lobby
Players gather in the lobby (3–10 players). The Host can start the game once at least 3 players have joined. Any player can leave by clicking "Back to Menu." If the Host leaves, the game is deleted.

### 3. Word Submission Phase
Once the game starts, every player privately submits **2 words or phrases** into a shared pool. The system also adds a number of **random words** (roughly equal to the number of players) from a built-in pool of 50 fun/quirky job titles and roles. This means no one can be sure whose word is whose.

### 4. Round Structure
The game consists of a fixed number of rounds equal to **2× the number of players** (e.g., 5 players = 10 rounds). Each player is assigned as Messenger exactly twice across all rounds.

### 5. Messenger Reveal
Each round, one player is randomly designated the **Messenger**. They see a word from the pool on their screen behind a blur overlay. They tap to reveal it, then **read it aloud** to the group at the table.

### 6. Discussion Phase
After the Messenger announces the word, all players discuss it at the table. The Host controls when to advance to the next round.

### 7. Game Over
After all rounds are complete, the game ends and all players see a "Game Over" screen. The game document is automatically deleted from the database.

---

## Core Features

| Feature | Description |
|---------|-------------|
| **Real-time Multiplayer** | Uses Firestore real-time listeners to keep all players perfectly in sync. |
| **Player-Generated Word Pool** | Each player contributes 2 words, making every game unique. |
| **System Random Words** | The game injects random words from a curated pool of 50 entries, so players can't be certain whose word is whose. |
| **Fair Messenger Rotation** | The Messenger role follows a pre-shuffled schedule ensuring every player is Messenger exactly twice. |
| **Reconnection Support** | Uses `localStorage` to detect and offer reconnection to in-progress games. |
| **Procedural Music** | Retro arcade soundtrack with 3 distinct tracks using Tone.js, rotating every 90 seconds. |
| **Retro Arcade Aesthetic** | Neon cyan/pink/green theme with animated grid background, scanlines, glitch effects, and pixel fonts. |
| **Separate Firebase Collection** | Uses `recruit_madness_games` collection — fully independent from other games on the same Firebase project. |

---

## Technical Architecture

The game is built as a **single-page application (SPA)** using a serverless architecture.

### Frontend
- **HTML**: A single `recruit-madness.html` file contains all game screens. Screens are toggled via CSS classes (`display: none` / `display: flex`).
- **CSS**: Custom CSS with CSS variables for the neon theme. No external CSS framework.
- **Fonts**: Orbitron (display), Press Start 2P (pixel/retro), Rajdhani (body) via Google Fonts.
- **JavaScript (ESM)**: All game logic in a single `<script type="module">` block — Firebase initialization, DOM manipulation, game state, and music engine.

### Backend (Serverless)
- **Google Firebase Firestore**: Real-time database. A single game document per session stores the entire state (players, word pool, round info, status). All clients subscribe via `onSnapshot`.
- **Firebase Authentication**: Anonymous sign-in gives each player a unique `userId` without account creation.
- **Collection**: `recruit_madness_games` (separate from any other games on the same Firebase project).

### Game State Machine
The game flow is controlled by the `status` field in the Firestore document:

```
lobby → submit_words → messenger_reveal → discussion → messenger_reveal → ... → (game deleted)
```

| Status | Description |
|--------|-------------|
| `lobby` | Players joining, waiting for Host to start |
| `submit_words` | Each player submits 2 words |
| `messenger_reveal` | Current Messenger sees the word, others wait |
| `discussion` | All players discuss at the table |

When all rounds are complete, the game document is deleted, which triggers all clients to return to the home screen.

---

## Setup & Installation

### Prerequisites
You need a Firebase project with:
1. **Firestore Database** enabled
2. **Anonymous Authentication** enabled

### Using an Existing Firebase Project
If you already have a Firebase project (e.g., from The Mole), you can reuse it. The game stores data in a separate collection (`recruit_madness_games`), so there's no conflict.

### Firebase Configuration
The `firebaseConfig` object in the HTML file must match your Firebase project:

```javascript
const firebaseConfig = {
    apiKey: "YOUR_API_KEY",
    authDomain: "YOUR_PROJECT.firebaseapp.com",
    projectId: "YOUR_PROJECT_ID",
    storageBucket: "YOUR_PROJECT.appspot.com",
    messagingSenderId: "YOUR_SENDER_ID",
    appId: "YOUR_APP_ID"
};
```

### Firestore Rules
Ensure your Firestore rules allow read/write on the new collection:

```
rules_version = '2';
service cloud.firestore {
  match /databases/{database}/documents {
    match /recruit_madness_games/{gameId} {
      allow read, write: if true;
    }
    // ... other collection rules
  }
}
```

> **Note:** `allow read, write: if true` is open access suitable for development/personal use. For production, add proper authentication checks.

---

## Deployment

### GitHub Pages (Recommended)
1. Add `recruit-madness.html` to your GitHub Pages repository.
2. If deploying alongside another game (e.g., The Mole), just place it in the same repo.
3. Access at `https://your-username.github.io/your-repo/recruit-madness.html`

### Any Static Host
The game is a single HTML file with no build step. Upload it to any static hosting service (Netlify, Vercel, Cloudflare Pages, etc.).

---

## Key Functions Explained

| Function | Description |
|----------|-------------|
| `createGame()` | Creates a new Firestore document in `recruit_madness_games` with initial state. |
| `joinGame(id, isNew)` | Connects a player to a game and sets up the real-time `onSnapshot` listener. |
| `addPlayer()` | Adds a player to the game's player array (with duplicate name check). |
| `startGame()` | Transitions from lobby to word submission phase (requires 3+ players). |
| `submitWord(word)` | Adds a word to the player's submissions in `wordSubmissions` map. |
| `finalizeWordPool()` | Called automatically when all players have submitted 2 words. Combines player words with random system words, shuffles, and creates the round order. |
| `getRandomSystemWords(count)` | Selects random entries from the built-in 50-word system pool. |
| `messengerDone()` | Messenger confirms they've read the word; transitions to discussion. |
| `nextRound()` | Advances to the next round with a new Messenger and word. Deletes game if final round. |
| `renderGame(data)` | Main UI router — reads `status` and calls the appropriate render function. |
| `setupMusic()` | Initializes Tone.js instruments and 3 retro arcade tracks with 90-second rotation. |

---

## System Word Pool

The game includes 50 built-in words that are randomly mixed into the player word pool each game. These are fun, quirky job titles and roles:

> Firefighter, Astronaut, Librarian, Archaeologist, Sommelier, Beekeeper, Cartographer, Gondolier, Blacksmith, Taxidermist, Puppeteer, Locksmith, Volcanologist, Chimney Sweep, Falconer, Cryptographer, Perfumer, Stunt Double, Ice Sculptor, Treasure Hunter, Storm Chaser, Snake Charmer, Sword Swallower, Town Crier, Lighthouse Keeper, Toy Designer, Food Critic, Train Conductor, Zookeeper, Parachute Instructor, Mime Artist, Submarine Captain, Diamond Cutter, Hot Air Balloon Pilot, Cheese Maker, Spy, Bounty Hunter, Forensic Scientist, Earthquake Engineer, Space Tourist, Robot Mechanic, Deep Sea Diver, Wildlife Photographer, Stuntman, Magician, Escape Room Designer, Voice Actor, Cave Explorer, Professional Gamer, Pet Psychic

The number of system words added equals the number of players in the game (minimum 2).

---

## Differences from The Mole

| Aspect | The Mole | Recruit Madness |
|--------|----------|-----------------|
| **Genre** | Social deduction (find the mole) | Word-pool party discussion |
| **Roles** | Word Master, Mole, Operatives | Messenger only |
| **Secret Info** | Mole doesn't know the word | Everyone hears the word |
| **Word Source** | Word Master types it each round | Player-submitted pool + system randoms |
| **Scoring** | No in-app scoring | No in-app scoring (external) |
| **Firebase Collection** | `games` | `recruit_madness_games` |
| **Visual Theme** | Dark spy aesthetic | Retro arcade / neon |
| **Music** | Spy-themed, 4 tracks | Retro arcade, 3 tracks |

---

## Browser Support
- Chrome, Edge, Safari, Firefox (latest versions)
- Mobile browsers (iOS Safari, Chrome for Android)
- Requires JavaScript enabled
- Requires internet connection for Firebase

---

## License
This project is intended for personal, non-commercial use only. All trademarks and registered trademarks are the property of their respective owners.
