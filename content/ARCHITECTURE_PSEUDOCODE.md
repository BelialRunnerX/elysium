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

---

## 2. Core Components

### 2.1 Position Component

```pseudo
struct Position {
    float x
    float y
}
```

**Explanation**: Almost every physical object in the game has a `Position` component. This allows the Movement System, Rendering System, and Collision System to all work on the same data.

---

### 2.2 Renderable Component

```pseudo
struct Renderable {
    string spriteName
    int layer
    bool visible
}
```

**Explanation**: Controls how an entity appears on screen. The `layer` helps with draw order (stars behind planets, UI on top, etc.).

---

### 2.3 Planet Component

```pseudo
struct Planet {
    float planetNess
    array<float, 20> mineralPercentages   // 20 sci-fi minerals
}
```

**Explanation**: Stores the core data that defines a planet. The `mineralPercentages` array is the source of truth for loot generation and resource systems.

---

### 2.4 Armor Component

```pseudo
struct Armor {
    string name
    int tier                    // 0 = Common ... 4 = Legendary
    float armorFlat
    float armorPercent
    float healthFlat
    float healthPercent
    float movementSpeedPercent
    int element                 // 0 = Void, 1 = Plasma, etc.
    array<Rune> runes           // 0–3 runes
}
```

**Explanation**: Armor is not a class — it is a component. Any entity (Player, NPC, Enemy) can have an `Armor` component. The stats are split into flat and percentage values so systems can apply them correctly.

---

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

**Explanation**: Similar to armor. The `element` field is critical for the combat system to determine bonuses against different armor types.

---

### 2.6 Psionic Component

```pseudo
struct Psionic {
    int element
    float power
    float cooldown
    string abilityName
}
```

**Explanation**: Psionics are treated as another form of "equipment" or ability. They follow the same elemental rules as weapons and armor.

---

## 3. Systems and Their Interactions

### 3.1 Combat System

```pseudo
function resolveAttack(attacker, target):
    if attacker has Weapon and target has Armor:
        weapon = attacker.get<Weapon>()
        armor = target.get<Armor>()
        
        damage = weapon.damageFlat * (1 + weapon.damagePercent)
        
        if weapon.element != armor.element:
            damage *= 1.25                    // Advantage
        
        if weapon.element == armor.element:
            damage *= 0.75                    // Disadvantage
        
        // Apply armor reduction
        effectiveArmor = armor.armorFlat * (1 + armor.armorPercent)
        finalDamage = max(1, damage - effectiveArmor)
        
        target.health -= finalDamage
```

**Explanation**: The combat system only runs on entities that have both `Weapon` and `Armor` components (or at least the attacker has a weapon). The elemental mismatch creates the advantage/disadvantage.

---

### 3.2 Loot Generation System

```pseudo
function generateLoot(planet):
    minerals = planet.get<Minerals>()
    
    for each mineral in minerals:
        if mineral.percentage > MINERAL_THRESHOLD:
            createLootItem(mineral)
```

**Explanation**: Loot is generated based on the minerals present on a planet. This ties the procedural generation directly to the item system.

---

### 3.3 Movement System

```pseudo
function moveEntity(entity, dx, dy):
    if entity has Position and not has Immobilized:
        pos = entity.get<Position>()
        pos.x += dx
        pos.y += dy
```

**Explanation**: The movement system only affects entities with a `Position` component. Effects like "Immobilized" are handled by adding/removing components.

---

## 4. Noise-Based Universe Generation

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

**Explanation**: The multi-dimensional noise allows us to layer different types of data (stars, planets, minerals) on the same coordinate space while keeping them independent.

---

## 5. Rendering System (2D Map)

```pseudo
function renderMap():
    for each entity with Position + Renderable:
        if entity has Star:
            drawStarSprite(entity.position)
        else if entity has Planet:
            drawPlanetSprite(entity.position, entity.get<Planet>().minerals)
```

**Explanation**: The rendering system queries for entities that have both `Position` and `Renderable`. It then checks what other components they have to decide how to draw them.

---

## 6. Condition-Based System Interaction (Hyperlinks)

In this architecture, **hyperlinks between components** represent conditions under which systems interact:

- `Weapon` + `Armor` → Combat System runs
- `Planet` + `Minerals` → Loot System runs
- `Position` + `Renderable` → Rendering System runs
- `Armor` + `Psionic` → Potential synergy or resistance checks

This is the core power of ECS — systems only activate when the required components are present.

---

*This document serves as the detailed pseudo code architecture for Elysium.*