# Photo Puzzle Game

## Overview

The Photo Puzzle Game is an interactive, retro-themed web application built with Vue.js, designed to celebrate a special occasion with a birthday-themed experience. The game features a central hub, the Arcade Universe, from which players can access multiple mini-games: a sliding tile Photo Puzzle, an Asteroid Love shooter, a Star Maze navigation game, and a Vault with a PIN-based unlock mechanism. The application incorporates nostalgic 8-bit aesthetics, neon visuals, and engaging gameplay mechanics, including easter eggs and a secret love letter.

## Features

- **Arcade Universe Hub**: A central menu allowing navigation to different game modes.
- **Photo Puzzle**: A sliding tile puzzle where players rearrange tiles to form a complete image. Difficulty levels range from Easy (2x2, 15 seconds) to Insane (5x5, 60 seconds).
- **Asteroid Love**: A space shooter where players control a ship to shoot hearts at asteroids, progressing through five levels.
- **Star Maze**: A maze navigation game where players collect hearts within a time limit, with an easter egg in level 4.
- **The Vault**: A PIN-entry screen to unlock a special love letter, using a 6-digit code based on a birthdate (e.g., 260902 for 26 September 2002).
- **Retro Aesthetics**: Features a CRT-style border, neon colors, and chiptune background music for an immersive retro experience.
- **Responsive Design**: Optimized for both mobile and desktop devices, with dynamic resizing for smaller screens.
- **Easter Eggs**: Includes a Konami code for a secret message and hidden love notes in the Arcade Universe.

## Prerequisites

To run the Photo Puzzle Game locally, ensure you have the following installed:

- Node.js (version 14 or higher)
- npm (version 6 or higher) or yarn
- A modern web browser (e.g., Chrome, Firefox, Edge)

## Setup Instructions

1. **Clone the Repository**:
   Run the following command in your terminal:
   ```bash
   git clone <repository-url>
   cd photo-puzzle-game
   ```

2. **Install Dependencies**:
   Using npm:
   ```bash
   npm install
   ```
   Or using yarn:
   ```bash
   yarn install
   ```

3. **Add the Image**:
   Place an image file named `angel.jpg` in the `public/` directory. This image is used for the sliding tile puzzle.
   Ensure the image is appropriately sized (e.g., 360x360 pixels for optimal display).

4. **Run the Application**:
   Start the development server using:
   ```bash
   npm run dev
   ```
   Or with yarn:
   ```bash
   yarn dev
   ```

5. **Access the Game**:
   Open your browser and navigate to `http://localhost:5173` (or the port specified by your setup).

## Dependencies

The project relies on the following key dependencies, listed in the `package.json`:

- Vue.js (^3.2.0): For building the reactive user interface.
- Vite (^4.0.0): For fast development and bundling.
- Tailwind CSS (optional, if used in the project): For styling, included via CDN in the provided code.

### External resources:

- **Google Fonts**: For the "Press Start 2P" and "Great Vibes" fonts.
- **Chiptune Audio**: Hosted at [https://cdn.pixabay.com](https://cdn.pixabay.com) for background music.

## Usage

1. **Starting the Game**:
   The game begins with a Birthday Special Screen displaying a celebratory message.
   Click "ENTER ARCADE UNIVERSE" to access the Arcade Universe hub.

2. **Navigating the Arcade Universe**:
   From the hub, select one of the game modes: Photo Puzzle, Asteroid Love, or Star Maze.
   Click on floating hearts to reveal hidden love notes.
   Enter the Konami code (↑↑↓↓←→←→BA) to unlock a secret message.

3. **Playing the Games**:
   - **Photo Puzzle**: Select a difficulty level (Easy, Normal, Hard, Insane) and click tiles to swap them, forming a complete image within the time limit (15 seconds for Easy, 60 seconds for others).
   - **Asteroid Love**: Use ←→ keys to move the ship and Space to shoot hearts at asteroids. Clear five levels to win.
   - **Star Maze**: Use arrow keys (↑↓←→) to navigate the maze and collect hearts within 60 seconds per level. Find the easter egg in level 4 for a special message.
   - **The Vault**: After completing the Photo Puzzle, enter the 6-digit PIN (e.g., 260902) to unlock a love letter.

4. **Returning to the Hub**:
   Each game screen includes a "Back to Hub" button to return to the Arcade Universe menu.

## File Structure

```
photo-puzzle-game/
  public/
    angel.jpg            Image for the Photo Puzzle
  src/
    App.vue              Main Vue component with game logic and UI
    assets/
      styles.css         Custom styles (if not using scoped styles)
  index.html             Entry point with Vue and CDN dependencies
  package.json           Project metadata and dependencies
  README.md              This file
```

## Notes

- The game requires an internet connection for loading Google Fonts and the chiptune audio.
- The PIN for the Vault is hardcoded to 260902 (26 September 2002). Modify the `checkVaultPassword` function in `App.vue` to change the PIN.
- Ensure the `angel.jpg` image is present in the `public/` directory, as the Photo Puzzle will not display correctly without it.
- The application is designed to be responsive, but optimal performance is achieved on screens wider than 360 pixels.

## Troubleshooting

- **Image Not Displaying**: Verify that `angel.jpg` is in the `public/` directory and correctly referenced in the code.
- **Audio Not Playing**: Check your browser's autoplay settings, as some browsers block audio without user interaction.
- **Game Not Loading**: Ensure all dependencies are installed and the development server is running. Check the console for errors.

## License

This project is unlicensed and intended for personal use. Modify and distribute as needed for non-commercial purposes.
# patricia
# patricia
