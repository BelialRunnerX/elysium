# Elysium – Core Architecture Pseudo Code

## 1. High-Level Architecture

```pseudo
Universe
├── Seed (uint64)
├── Noise Generator (Multi-dimensional)
├── Star Entities
├── Planet Entities
└── Mineral Data per Planet
```

## 2. Noise Generation

```pseudo
function generateUniverse(seed):
    noise = FastNoise(seed)
    
    for x in universeWidth:
        for y in universeHeight:
            starValue   = noise.GetValue(x, y, 0)           // Star-ness
            planetValue = noise.GetValue(x, y, 1)           // Planet-ness
            
            if starValue > STAR_THRESHOLD:
                createStar(x, y, starValue)
            
            if planetValue > PLANET_THRESHOLD:
                createPlanet(x, y, planetValue)
```

## 3. Planet Generation

```pseudo
function createPlanet(x, y, planetNess):
    entity = registry.create()
    
    registry.emplace<Position>(entity, x, y)
    registry.emplace<Planet>(entity, planetNess)
    
    minerals = {}
    for i in 0..20:
        minerals[i] = noise.GetValue(x, y, 2 + i)
    
    registry.emplace<Minerals>(entity, minerals)
```

## 4. Rendering Logic (2D Map)

```pseudo
function renderMap():
    for each tile in map:
        if tile.starNess > STAR_THRESHOLD:
            drawStarSprite(tile.x, tile.y)
        
        if tile.planetNess > PLANET_THRESHOLD:
            drawPlanetSprite(tile.x, tile.y, tile.minerals)
```

## 5. Entity Component System Structure

### Core Components

```pseudo
struct Position {
    float x, y
}

struct Star {
    float brightness
    string spectralType
}

struct Planet {
    float planetNess
    array<float, 20> mineralPercentages
}

struct Minerals {
    map<int, float> percentages   // mineralID -> percentage
}

struct Renderable {
    string spriteName
    int layer
}
```

### Example Entity Creation

```pseudo
entity = registry.create()
registry.emplace<Position>(entity, x, y)
registry.emplace<Planet>(entity, planetNess)
registry.emplace<Minerals>(entity, mineralData)
registry.emplace<Renderable>(entity, "planet_icon", 1)
```

## 6. Threshold System

```pseudo
STAR_THRESHOLD   = 0.75
PLANET_THRESHOLD = 0.65

MINERAL_THRESHOLDS = {
    0: 0.4,   // Titanium
    1: 0.3,   // Quantium
    2: 0.5,   // Void Crystal
    ...
}
```

## 7. Future Expansion (Not in Prototype)

- Player Entity + Inventory
- Combat System (ECS-based)
- Loot Generation tied to minerals
- Psionic abilities as components
- Faction influence system

---

*This document serves as the living pseudo code architecture for Elysium.*