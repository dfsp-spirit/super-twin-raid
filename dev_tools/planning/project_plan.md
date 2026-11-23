# Project Plan for Super Twin Raid (STR)

## Goals: Develop a Silkworm-like 2D shmup for PC, with support for local coop (PvE).

## Properties:

* The entire game consists of 2D sprites, it is not 2.5D and has no support for displaying meshes or other 3D assets, no dynamic lighting, etc.
* It should be easy to generate distribution-ready versions for PC (x86, x64).
* The game is a hobby project, and it is a priority to get to a testable version early, to prevent lots of wasted effort with developing too many complex systems too early that may turn out not to be fun and may need to get replaced
* It should use an established game engine, like Godot
* We do not intend to invest (lots of) money into store assets or other things. We prefer to make our own, simple assets (sprites, sounds). For the prototype, using placeholders like art with an open license is fine, but in the end we want a consisten art style, not a collection of random assets. Looks are important to us.
* asymmetrical gameplay option: one can play in helicopter or jeep, and in coop, one of each. The levels are designed in a way that it is possible to finish them alone and the jeep or the helicopter.


## Required Features:

* Input support: Game controllers (e.g., Xbox One controller for PC), and alternatively keyboard. It should be possible to play with 2 people on one keyboard for people who do own any controllers.
* Pre-made levels, with hard-coded positions for enemies (including rows of them flying in typical shmup patterns).
* Upgrades: only during a run: upgrades like double-shot that are lost on hit
* Different enemy types, similar to SW:
    - different helicopters, fighter jets, tanks, jeeps, mines, obstacles
    - enemies attack with:
        * no weapons at all (like fast jets, mines, large rockets going straight up from the ground out of missile tubes, ground obstacles which are a threat mainly because you can crash into them)
        * guided missiles, that can be destroyed by player fire
        * unguided missiles, that cannot be destroyed by player fire
* Shields that give temporary invulnerability, but can also alternatively be shot with a few hits to explode, dealing massive damage to all enemies on screen. Exploding a shield also explodes all other shield powerups currently on the screen (but not shields which the player already has active)
* Collisions kill the player, unless they have a shield active, in which case the player deals high damage in short time intervalls to enemies they touch
* Killing enemies increases ones score
* Players have a (shared in coop) limited number of lives, and a number of continues (configurable, in settings). Using a continue resets your score to 0.
* The game should prevent the annoying shmup trap where a player is lost because they were hit once and now do not have the double-shot (or other powerups) anymore, and this lack of firepower leads to inevitable death later. Preventing such issues can be achieved, for example, by providing again the double-shot power-up before bosses or hard parts of a map. Picking up a power-up you already have should give a score bonus instead.


## Explicitely out of scope:

* Non-local multiplayer over the internet
* Rogue-lite progression with permanent upgrades
* 2.5D or 3D, support for meshes, lighting, etc. It's strictly a 2D sprite-based game.