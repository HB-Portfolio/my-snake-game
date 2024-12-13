# My Snake Game

## Project Background
This project represents my first full integration of custom-designed assets into a web game. Building on my previous junior game development experience with Asteroid Field (Python) and Block Stacking Game (Vanilla JS), this project pushed me to combine both design and development skills in a new way.

## Development Goals vs Reality
Starting from the design system created in Project 2, the main challenge was turning static assets into a playable game. Here's how the implementation matched up with the original goals:

### What Was Implemented
- **Complete Asset Integration**: Successfully implemented all sprites and UI elements from Project 2's design system
- **Core Game Mechanics**:
  - Smooth snake movement
  - Collision detection
  - Growth mechanics
  - Scoring system with multiple collectible types
- **Cross-Device Compatibility**:
  - Desktop keyboard controls
  - Mobile touch controls with custom button layout
  - Responsive game board that maintains playability across screen sizes

### Major Technical Challenges
1. **Firefox Compatibility Crisis**
   - During deployment, discovered Tailwind's `max-w` classes were being parsed as route parameters with '&'
   - Had to completely restructure width management across the entire game
   - Required rapid problem-solving under deadline pressure

2. **First-Time Asset Integration**
   - First project integrating custom-designed sprites into actual gameplay
   - Required careful management of sprite positioning and transitions
   - Learned valuable lessons about asset optimization and implementation

### Scope Management Decisions

#### Features Intentionally Omitted
1. **Supabase Integration**
   - Originally planned for leaderboard functionality
   - Decided against implementation because:
     - Didn't want to force users to log in to play
     - Wanted to keep the game instantly accessible
   - Future plan: Implement arcade-style 3-character name leaderboard instead

2. **Colorblind Filters**
   - Originally considered in accessibility planning
   - Postponed due to:
     - Time constraints
     - First-time asset integration taking priority
     - Need to thoroughly test filter effects on custom sprites

## Technical Implementation

### Core Technologies
- **Framework**: Nuxt 3
- **Styling**: Tailwind CSS (with custom configuration for game-specific needs)
- **Asset Management**: Custom sprite system

### Key Components
```
GameComponent.vue
└── Core game logic
    ├── Movement system
    ├── Collision detection
    ├── Scoring mechanics
    ├──  Sprite management
    └── Cross-device controls

index.vue
└── Game container
    ├── Background management
    └── Responsive layout
```

### Development Timeline
Actual time spent vs planned (40-hour total):
- Game Logic & Movement: 15 hours (as planned)
- Asset Integration: 12 hours (2 hours over due to Firefox bug)
- Scoring System: 4 hours
- Bug Fixing: 7 hours (2 hours over due to width class issues)
- Final Polish: 2 hours (reduced to accommodate bug fixing)

## Running the Game

```bash
# Install dependencies
npm install

# Development server
npm run dev

# Production build
npm run build
```

## Future Roadmap
1. **Immediate Plans**:
   - Implement arcade-style leaderboard (3-character names, no login required)
   - Add basic sound effects
   
2. **Later Considerations**:
   - Colorblind accessibility options
   - Additional game modes
   - Power-up system

## Learning Outcomes
1. First successful integration of custom-designed assets into a functional game
2. Gained experience in scope management and feature prioritization
3. Learned to handle critical cross-browser compatibility issues
4. Improved understanding of asset optimization for web games

## Known Issues
- Touch controls could benefit from additional optimization
- Need to implement better fallback patterns for failed sprite loads

*Note: This game represents my growth from pure game logic implementation (as in my previous projects) to full-stack game development including custom assets and responsive design.*
