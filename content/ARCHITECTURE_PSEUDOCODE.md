# Elysium – Core Architecture Pseudo Code (Explained)

## 1. High-Level Architecture

**Core Idea**: Everything in the game is built using an **Entity Component System (ECS)**. There are no traditional classes for "Player", "Planet", or "Armor". Instead, entities are just IDs that hold collections of components.

```pseudo
Universe
├── Seed (uint64)
├── Noise Generator (Multi-dimensional)
├── Entities (Stars, Planets, Player, Loot, etc.)
└── Systems (Movement, Combat, Loot, Rendering, etc.)
```

**Key Principle**: Systems operate on entities that have the required components. This allows maximum flexibility and decoupling.

## 2. Core Components

### 2.1 Position Component

```pseudo
struct Position {
    float x
    float y
}
```

**Explanation**: Almost every physical object in the game has a `Position` component. This allows the Movement System, Rendering System, and Collision System to all work on the same data.

### 2.2 Renderable Component

```pseudo
struct Renderable {
    string spriteName
    int layer
    bool visible
}
```

**Explanation**: Controls how an entity appears on screen. The `layer` helps with draw order.

### 2.3 Planet Component

```pseudo
struct Planet {
    float planetNess
    array<float, 20> mineralPercentages
}
```

**Explanation**: Stores the core data that defines a planet. The `mineralPercentages` array is the source of truth for loot generation.

### 2.4 Armor Component

```pseudo
struct Armor {
    string name
    int tier
    float armorFlat
    float armorPercent
    float healthFlat
    float healthPercent
    float movementSpeedPercent
    int element
    array<Rune> runes
}
```

**Explanation**: Armor is a component. Any entity can have it. Stats are split into flat and percentage values.

### 2.5 Weapon Component

```pseudo
struct Weapon {
    string name
    int tier
    float damageFlat
    float damagePercent
    float fireRate
    int element
}
```

**Explanation**: Similar to armor. The `element` field is critical for combat calculations.

### 2.6 Psionic Component

```pseudo
struct Psionic {
    int element
    float power
    float cooldown
    string abilityName
}
```

**Explanation**: Psionics are treated as another form of ability/equipment.

## 3. Component Attachment Rules

- **Irrelevant components** are **not attached**.
- **Optional components** may be attached but can be set to `null` when not in use.

## 4. Systems and Their Interactions

### 4.1 Combat System

```pseudo
function resolveAttack(attacker, target):
    if attacker has Weapon and target has Armor:
        // Calculate damage with elemental modifiers
        // Apply armor reduction
        // Apply critical hits if applicable
```

### 4.2 Loot Generation System

```pseudo
function generateLoot(planet):
    minerals = planet.get<Minerals>()
    for each mineral in minerals:
        if mineral.percentage > MINERAL_THRESHOLD:
            createLootItem(mineral)
```

### 4.3 Movement System

```pseudo
function moveEntity(entity, dx, dy):
    if entity has Position and not has Immobilized:
        pos = entity.get<Position>()
        pos.x += dx
        pos.y += dy
```

## 5. Noise-Based Universe Generation

```pseudo
function generateUniverse(seed):
    noise = FastNoise(seed)
    for x in universeWidth:
        for y in universeHeight:
            starValue   = noise.GetValue(x, y, 0)
            planetValue = noise.GetValue(x, y, 1)
            
            if starValue > STAR_THRESHOLD:
                createStarEntity(x, y, starValue)
            
            if planetValue > PLANET_THRESHOLD:
                createPlanetEntity(x, y, planetValue)
```

## 6. Rendering System (2D Map)

```pseudo
function renderMap():
    for each entity with Position + Renderable:
        if entity has Star:
            drawStarSprite(entity.position)
        else if entity has Planet:
            drawPlanetSprite(entity.position, entity.get<Planet>().minerals)
```

## 7. Condition-Based System Interaction

In this architecture, the presence or absence of components determines which systems affect an entity. This creates a natural "hyperlink" between components and systems.

See also: [[GAME_ARCHITECTURE]], [[Player_Character]], [[Armor]], [[Weapon]], [[Psionic]]