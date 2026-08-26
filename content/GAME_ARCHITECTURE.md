# Elysium – Game Architecture

## Overview

Elysium is built on an **Entity Component System (ECS)** architecture. There are no traditional classes such as `Player`, `Planet`, or `Armor`. Instead, every object in the game is an **Entity** (an ID) that possesses zero or more **Components**. **Systems** operate on entities that have the required components.

This design allows for maximum flexibility, data-oriented performance, and clean separation of concerns.

See also: [[ARCHITECTURE_PSEUDOCODE]], [[ECS_Overview]]

## Core Principles

- **Entities** are simple IDs.
- **Components** hold all data (Position, Health, Armor, Weapon, etc.).
- **Systems** contain all logic (Movement System, Combat System, Loot System, Rendering System, etc.).
- Systems only operate on entities that have the required components.

## Component Attachment Rules

- **Irrelevant components** are **not attached** to an entity.
- **Optional components** may be attached but can hold a `null` value when not in use.

## Major Entity Types

- [[Player_Character]]
- [[NPC]]
- [[Planet]]
- [[Star]]
- [[Structures]]

## Major Component Categories

- **Core Components**: Position, Health, Power, Stats
- **Equipment Components**: [[Armor]], [[Weapon]], [[Psionic]]
- **World Components**: [[Minerals]], [[Resources]]
- **Progression Components**: Inventory, Scriptures

## Major Systems

- [[Combat_System]]
- [[Loot_System]]
- [[Movement_System]]
- [[Rendering_System]]

## World Generation

The universe is generated from a seed using multi-dimensional noise. Stars, planets, and minerals are placed based on threshold values.

See also: [[Universe_Generation]]