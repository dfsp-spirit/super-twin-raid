# Project Plan for Super Twin Raid (STR)

## Goals: Develop a Silkworm-like 2D shmup for PC, with support for local coop (PvE).

## Properties:

* The entire game consists of 2D sprites, it is not 2.5D and has no support for displaying meshes or other 3D assets, no dynamic lighting, etc.
* It should be easy to generate distribution-ready versions for PC (x86, x64). If feasible without much extra effort, a web-based version would be a nice extra.
* The game is a hobby project, and it is a priority to get to a testable prototype version early, to prevent lots of wasted effort with developing too many complex systems too early that may turn out not to be fun and may need to get replaced
* It should use an established game engine, like Godot
* We do not intend to invest (lots of) money into store assets or other things. We prefer to make our own, simple assets (sprites, sounds). For the prototype, using placeholders like art with an open license is fine, but in the end we want a consisten art style, not a collection of random assets. Looks are important to us.


## Required Features:

* Auto-scrolling game world in 2D, it never stops, not even at bosses.
* Asymmetrical gameplay option: one can play in helicopter or jeep, and in coop, one of each. The levels are designed in a way that it is possible to finish them alone and the jeep or the helicopter. We may need to mark enemies as coop-only to adjust difficulty, maybe even heli-only or jeep-only in single-player later. Or adapt difficulty in other ways, e.g., scale enemy health. Vehicle is fixed for entire run once selected.
* Player Movement:
    - Both the heli and the jeep can move left and right (forward and backwards, from their own perspective) within the moving field of view.
    - The heli can also move up and down (including 45 degrees down/up)
    - The jeep can jump, allowing it to evade rockets and jump over fast suicide-car style enemies or anti-vehicle mines
    - The ground is always completely flat, so there is no need for terrain following for the jeep.
    - The jump high of the jeep is low, and the heli has a minimal height, so actually they cannot even overlap / collide. Whether you play alone or in coop also thus does not affect the space you have to dodge enemy attacks in any way.
* Enemy movement:
    - Enemies can just go off-screen or stay on screen forever, depending on enemy type (like suicide cars trying to hit jeep player, or fixed stuff like ground turrets, they will just leave screen if not destroyed).
    - Some enemies may linger around, i.e., fly/drive backwards (from their perspective) with scroll speed, i.e., just stand still relative to the pixel position on the monitor. (But they can only hang out in this way at the right part of screen, near their entry point: neither of the players can shoot backwards) if not destroyed. In any case, it never happens that screen/world is stuck while the game waits for player to clear a wave or for other reasons, not even on death.
* Player death: the player respawns quickly (the world keeps scrolling), causing damage that destroys basic enemies in a medium area around it, and is invulnerable for a few seconds after respawning. They respawn where they died.
* In coop, both players see the SAME screen all the time: its not split-screen, its shared screen.
* Player weapons that enable a single player to still complete the maps:
    - the Heli has a powerful weapon that shoots straight forward (a line of missiles, or with upgrade 2 lines, that acts almost like a beam: its missile after missile) AND a small gun that fires down and forward at a fixed 45 degree angle
    - the Jeep has a single cannon in the back that should a similar stream of bullets (2 streams with powerup). But using 2 keys for gun-up and gun-down, the weapon can be aimed: from straight up to parallel to the ground, in front of the jeep.
    - For the heli, the powerup just adds a parallel stream of front missiles, and does not affect the 45 degrees gun. for the jeep, the powerup adds a 2nd stream of bullets to the aimable gun, however not in parallel, but with a slight angle of about 5 to 10 degrees, so the jeep can cover a slighlty whider area with the upgrade (to make aiming at trash targets easier), but the angle is low so that larger, powerful enemies like bosses can still be hit with both streams of bullets at once.
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
* Collisions kill the player, unless they have a shield active, in which case the player deals high damage in short time intervalls to enemies they touch. In coop, the heli and jeep cannot collide (due to different space/heli min flying height) or hurt each other
* Killing enemies increases ones score
* Players have a (shared in coop) limited number of lives, and a number of continues (configurable, in settings). Using a continue resets your score to 0 and restarts the current level.
* The game should prevent the annoying shmup trap where a player is lost because they were hit once and now do not have the double-shot (or other powerups) anymore, and this lack of firepower leads to inevitable death later. Preventing such issues can be achieved, for example, by providing again the double-shot power-up before bosses or hard parts of a map. Picking up a power-up you already have should give a score bonus instead.
* It would be very nice to have an easy-to-use level editor, maybe even campaign editor (just a list of levels with ordering).
* One should be able to unlock something by finishing levels. Not permanent upgrades, but maybe a different skin for your vehicle or something? Need to think about this a bit more.
* Amount of content:
    - 5 levels: Greenish/temperate area, seascape, yellow desert, brown desert, night in the mountains.
    - Each has a few minutes of play time and they introduce new harder enemies (but older enemies also can occur, so variety also increases).
    - Each level features a boss at the end.
* Scoring: in coop, the player who gets the last hit (killing blow) on an enemy gets the score, except for bosses its always shared.
* HUD: minimal, at the top of screen only, not over the play area. Shows number of lifes, score, shield/invulnerability state, upgrades (double-shor or not). The shields are also clearly visible in-game as a cloud around player, and the double-shot from seeing your shots, so the HUD is only very rarely consulted by players.
* Pause button.
* Game speed independent of frame rate. Frame rate cap to save power/noise/heat, in menu.
* Controls, with the following defaults:
    - Keyboard:
        * Jeep:
            * a: move left
            * d: move right
            * w: jump
            * left shift: cannon angle up
            * left ctrl: cannon angle down
            * <|> (the key between left shift and y): shoot
            * b: alternative for cannon angle up
            * n: alternative for cannon angle down
            * m: alternative for shoot
        * Helicopter
            * arrow-up: fly up
            * arrow-down: fly down
            * arrow-left: fly left
            * arrow-right: fly right (combinations like up+right = 45 degrees up are possible)
            * right control: shoot
        * General:
            * pause key: pause/continue
            * ESC: open menu (pauses as well)
            * menu controls:
                * wasd and arrow keys possible. the 2 middle keys navigate up/down in menu.
                * the right-most key (right-arrow/d) is confirm/enter menu level (alternative key: Enter)
                * the left-most key (left-arrow/a) is leave menu level (alternative key: escape)
                * if paused, the menu has a continue button
    - Game Controller
        * Jeep
            * left DPAD or left stick: move left/right.
            * right trigger or A (lower of the 4 buttons on the right): shoot
            * Y (left of the 4 buttons on the right): jump
            * left/right shoulder button: change cannon angle (left: up, right: down/to front)
        * Helicopter:
            * left DPAD or left stick: move
            * right trigger or A (lower of the 4 buttons on the right): shoot
        * General:
            * menu button: menu

## Prototype
* 1 level (hand-built), with placeholder graphics and sounds
* Default key / control layout only

## Required only at late stage in development
* More complex goose miniboss with animation that gets constructed in the air out of several flying parts
* Proper sounds, music, etc. But we need some placeholder sounds for the prototype.
* Level editor
* Rebindable controls


## Explicitely out of scope:

* Non-local multiplayer over the internet
* Rogue-lite progression with permanent upgrades
* 2.5D or 3D, support for meshes, lighting, etc. It's strictly a 2D sprite-based game.
* Floor as curves, and the thus the need for the jeep to follow special terrain. The ground is always flat.