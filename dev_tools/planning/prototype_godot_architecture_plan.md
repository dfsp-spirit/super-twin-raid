# Super Twin Raid (STR) — Godot 4 Prototype Architecture

## 1. Project & Scene Hierarchy

Keep the prototype flat and decoupled. The key architectural trick for an auto-scrolling shmup where the world never stops is to **move a `Camera2D` steadily to the right** while keeping player coordinates relative to a moving playfield frame.

```text
Main (Node)
├── GameWorld (Node2D)
│   ├── ParallaxBackground
│   │   ├── SkyLayer (ParallaxLayer)
│   │   └── GroundLayer (ParallaxLayer - visual floor)
│   ├── ScrollingStage (Node2D)
│   │   ├── WaveSpawner (Node2D) <-- Holds placed EnemySpawnMarker2D nodes
│   │   └── ActiveEntities (Node2D) <-- Spawned enemies, pickups, explosions live here
│   ├── Projectiles (Node2D) <-- Separate container so bullets don't inherit enemy transforms
│   └── PlayfieldFrame (Node2D) <-- Moves right at SCROLL_SPEED alongside Camera2D
│       ├── Camera2D
│       ├── ScreenBounds (StaticBody2D) <-- Keeps players inside the visible viewport
│       └── Players (Node2D)
│           ├── JeepPlayer (CharacterBody2D)
│           └── HeliPlayer (CharacterBody2D)
└── HUD (CanvasLayer) <-- Top bar only (lives, score, active upgrade/shield icons)
```

### Why Moving `Camera2D` Right is Cleanest
* **Direct Level Layout:** Place enemies directly in the Godot 2D level editor at their exact world `X` coordinates (e.g., `x = 2400`, `x = 4800`).
* **Synchronized Playfield:** Move `PlayfieldFrame` (holding `Camera2D`, `ScreenBounds`, and `Players`) to the right at `SCROLL_SPEED` (e.g., `150 px/s`).
* **Simple Lingering Enemies:** Lingering air enemies simply set their `velocity.x = SCROLL_SPEED` once they reach their target offset near the right edge of the viewport—no complex relative coordinate math required.
* **Automatic Off-Screen Cleanup:** Attach a `VisibleOnScreenNotifier2D` to every enemy and projectile. Connect its `screen_exited` signal to `queue_free()` (with a small margin on the right so newly spawned enemies aren't deleted before entering).

---

## 2. Collision Layers & Masks (Godot 4 Bitmask Setup)

Godot 4 provides 32 physics layers. Defining the first 8 cleanly enforces your core gameplay rules (such as the Jeep and Helicopter never colliding, and guided missiles being shootable while unguided ones pass through bullets) directly in the physics engine without extra conditional checks.

| Layer # | Name | Who lives on this layer? |
| :--- | :--- | :--- |
| **1** | `world_floor` | Flat ground `StaticBody2D` (stops Jeep gravity/jump) |
| **2** | `player_jeep` | Jeep `CharacterBody2D` + pickup `Area2D` |
| **3** | `player_heli` | Helicopter `CharacterBody2D` + pickup `Area2D` |
| **4** | `player_bullets` | Jeep cannon rounds, Heli front missiles & 45° gun shots |
| **5** | `enemies` | All enemy vehicles, turrets, mines, and bosses |
| **6** | `enemy_unguided` | Unguided rockets, bullets, laser beams (cannot be shot down) |
| **7** | `enemy_guided` | Homing/guided missiles (can be destroyed by `player_bullets`) |
| **8** | `pickups` | Shields, double-shot power-ups, extra lives |

### Collision Mask Matrix (What Detects What)

* **`player_jeep` (Layer 2):**
  * **Mask:** `1` (`world_floor`), `5` (`enemies`), `6` (`enemy_unguided`), `7` (`enemy_guided`), `8` (`pickups`).
  * *Behavior:* Does **not** mask Layer 3 (`player_heli`). They physically cannot collide in co-op.
* **`player_heli` (Layer 3):**
  * **Mask:** `5` (`enemies`), `6` (`enemy_unguided`), `7` (`enemy_guided`), `8` (`pickups`).
  * *Clamping:* Enforce `global_position.y <= MIN_HELI_ALTITUDE` in `_physics_process()` so the Helicopter stays strictly above the Jeep's maximum jump apex.
* **`player_bullets` (Layer 4):**
  * **Mask:** `5` (`enemies`), `7` (`enemy_guided`), `8` (`pickups` — specifically so bullets hit uncollected shields).
  * *Pickup Filtering:* Non-shield pickups (like double-shot or extra lives) ignore `player_bullets` or no-op on hit, whereas an uncollected Shield pickup takes damage (after a short spawn grace period) and triggers its screen-wipe explosion at `hp <= 0`.
* **`enemy_unguided` (Layer 6):**
  * **Mask:** `2` (`player_jeep`), `3` (`player_heli`).
  * *Behavior:* Player bullets pass right through unguided missiles without destroying them.
* **`enemy_guided` (Layer 7):**
  * **Mask:** `2` (`player_jeep`), `3` (`player_heli`), `4` (`player_bullets`).
  * *Behavior:* Can damage players *and* be shot down by player fire.

---

## 3. Reusable Node Blueprints for the Prototype

### Player Vehicle (`CharacterBody2D`)
* `Sprite2D`: Placeholder rectangle/silhouette. For the Jeep, add a child `TurretPivot (Node2D)` that rotates between `0°` (horizontal forward) and `-90°` (straight up).
* `CollisionShape2D`: Slightly smaller than the visual sprite (standard shmup "forgiving hitbox" design).
* `Hurtbox (Area2D)`: Monitors Layers `5, 6, 7`. Triggers `die()` unless `has_shield` or `is_invulnerable` is active.
* `ShieldAura (Area2D)`: Disabled by default; enabled when a shield pickup is active. Monitors Layer `5` (`enemies`) and deals rapid tick damage on contact.
* `MuzzleMarker (Marker2D)`: Child of `TurretPivot` (for Jeep) so projectiles always spawn cleanly at the barrel tip regardless of aim angle.

### Level Spawner (`EnemySpawnMarker2D` extends `Marker2D`)
Instead of building a custom in-game level editor for the prototype, create a lightweight `EnemySpawnMarker2D` script with `@export` properties:
* `@export var enemy_scene: PackedScene`
* `@export var spawn_mode: SpawnMode` (`ALWAYS`, `COOP_ONLY`, `SOLO_ONLY`)
* `@export var linger_duration: float = 0.0` (Use `-1.0` for bosses that linger indefinitely)

Place these markers directly inside your prototype level scene in the Godot 2D viewport. When `Camera2D`'s right edge passes a marker's `global_position.x`, it checks `spawn_mode` against the active player count, instantiates `enemy_scene` into `ActiveEntities`, passes `linger_duration`, and frees the marker.

---

## 4. Prototype Folder Structure

```text
res://
├── autoload/
│   ├── GameState.gd       # Lives, continues, scores, coop mode flag, auto-fire setting
│   └── Events.gd          # Global signal bus (enemy_killed, shield_detonated, player_died)
├── entities/
│   ├── players/           # Jeep.tscn, Heli.tscn, PlayerBase.gd
│   ├── enemies/           # EnemyBase.gd + 3-4 prototype enemies (Turret, Mine, Jet, Boss)
│   ├── projectiles/       # Bullet.tscn, GuidedMissile.tscn, UnguidedMissile.tscn
│   └── pickups/           # ShieldPickup.tscn, DoubleShotPickup.tscn
├── levels/
│   ├── LevelBase.tscn     # Camera, PlayfieldFrame, Floor, HUD
│   └── Level01_Proto.tscn # Inherits LevelBase, contains placed EnemySpawnMarkers
└── ui/
    └── HUD.tscn
```

Using a lightweight `Events.gd` autoload signal bus keeps the prototype decoupled and fast to iterate on: when a shot shield explodes, it emits `Events.shield_detonated.emit()`, allowing all active enemies and uncollected shields on screen to react cleanly without hard references between nodes.