# Ocarina of Time (US NTSC 1.0) cheat to-do list

Proposed OoT versions of the ALTTP cheats tagged `demo-ready`
(`snes/AsarUSALTTPDisassembly/game_genie/alttp_game_genie.md` and
`alttp_game_genie_long_codes.md`). The ALTTP title is in parentheses when
it differs. Source pointers are starting points, not checked fixes.
Checked items have an entry in `oot_patches.md`.

ALTTP cheats left out because OoT has nothing like them: Pegasus Boots
dash (dash without boots, steer while dashing, dash starts at once), Magic
Cape, bunny Link (both), Zelda following Link, pull a bush or pot out of a
wall, good bees (both), the Lamp lets out a bee, every sprite acts like a
soldier, enemies die like a boss, tutorial soldiers go through 256
messages, the "BAGE" debug name (folded into the debug save item below),
sword beams fire at any health, and the two fireball long codes.

- [x] Adult Link rides his horse backwards
- [x] Enable `DebugDisplay_*` object display

## Movement

- [x] Link runs twice as fast (Link walks twice as fast). `R_RUN_SPEED_LIMIT`
  and `Player_GetMovementSpeedAndYaw` in `ovl_player_actor/z_player.c`.
- [x] Link walks through walls. Skip or neuter the wall part of
  `Player_ProcessSceneCollision` / `Actor_UpdateBgCheckInfo` for the player.
- [x] All floors are slippery like ice. Return the ice floor property
  from the surface-type lookup in `z_bgcheck.c`.

## Speed and time

- [x] The game runs at half speed. Double the update-rate divisor
  (`R_UPDATE_RATE`).
- [x] The game runs faster than normal. (whatever amount is practical without soft-locking)
- [x] Frame advance on controller 1 (L pauses the game, R steps one frame).
  `FrameAdvance_Update` (`src/code/z_frame_advance.c`) is in the retail
  build but reads controller 2 (R + D-Down, then Z + R). Point it at
  `input[0]` in `Play_Update`, or change the buttons.
- [x] Time moves only while any button or the analog stick is being pressed.
  Use `FrameAdvance_Update`.
- [x] No delay for opening treasure chests. Shorten the chest-open wait
  in `ovl_En_Box` and the player's chest-open action.
- [x] Link picks up pots, bushes and rocks with almost no delay. Speed up
  the player's lift action.

## Debug and start state

- [x] Start a new game with the debug save (Start a new game with BAGE
  debug loadout). `Sram_InitDebugSave` (`src/code/z_sram.c`) is built
  into retail. Retail calls it only for the title-screen demo
  (`ovl_opening/z_opening.c`); the call for file 1 in `Sram_InitSave` is
  behind `DEBUG_FEATURES`. Call it from `Sram_InitSave` instead of
  `Sram_InitNewSave`.
- [x] Turn on the map select (Turn on the controller 2 debug features).
  `ovl_select` is in the retail ROM. Boot into it, or open it from the
  title screen.
- [x] Start a new game with a better sword and shield (Link's uncle gives
  the Master Sword). Child Link can't hold the Master Sword, so give the
  Kokiri Sword and Deku Shield from the start.
- [x] Link's house door leads to an unused or debug room (Link's house
  door leads to the Chris Houlihan room). Change the entrance index of
  Link's house exit to a test map.

## Combat

- [x] Every enemy dies in one hit. Raise the damage the player's attacks
  deal, or zero enemy health on any hit.
- [x] Spin attack charges at once. Start the spin charge counter at full.
- [x] Hold B to swing the sword again and again with no delay. Let the
  sword action restart while B is held.
- [x] Sword hits enemies across most of the screen. Make the sword
  collider (`meleeWeaponQuads`) much bigger.
- [x] Shield blocks shots from any direction. Skip the facing check in
  the player's shield/guard collision.
- [x] Thrown objects hit enemies anywhere. Make the collider of thrown
  pots, rocks and bombs much bigger.
- [x] Arrows fly through enemies (Sword beams fly through enemies).
  `ovl_En_Arrow`: don't kill the arrow when it hits an actor.
- [x] Arrows fly twice as fast (Sword beams fly twice as fast).
  `ovl_En_Arrow`.
- [x] Arrows fly at quarter speed (Sword beams fly at quarter speed).
  `ovl_En_Arrow`.
- [ ] Arrows stay where they appear (Sword beams stay where they appear).
  `ovl_En_Arrow`.
- [ ] Every arrow is an ice arrow (The sword beam is an ice rod shot).
  `ovl_En_Arrow` / `ovl_Arrow_Ice`.
- [ ] Every arrow is a fire arrow (The sword beam is a fire rod shot).
  `ovl_En_Arrow` / `ovl_Arrow_Fire`.
- [ ] A Deku Nut casts Din's Fire (The Lamp casts Bombos). Spawn the
  Din's Fire actor (`ovl_Magic_Fire`) where the Deku Nut is thrown.
- [ ] Every sword swing makes a spin-attack magic wave (The Fighter's
  Sword shoots beams). Spawn the great-spin effect on each swing.
- [ ] Nayru's Love is always on (The Magic Cape is always on). Keep the
  Nayru's Love timer from running down, or start it on.

## Damage

- [ ] Enemies don't knock Link back. Skip the knockback in the player's
  damage handling (`func_80837C0C` and nearby in `z_player.c`).

## Enemies

- [ ] Enemies always drop a prize. `Item_DropCollectibleRandom`
  (`src/code/z_en_item00.c`): skip the "no drop" roll.
- [ ] Enemies drop cuccos instead of prizes. Spawn `ovl_En_Niw` from
  `Item_DropCollectibleRandom`.
- [ ] Enemies face and walk away from Link. Flip the yaw-to-player in the
  common enemy chase code, or in `Actor_WorldYawTowardActor`.
- [ ] Enemies pass through Link. Turn off enemy-to-player collision
  (OC) in `z_collision_check.c`.
- [ ] Enemies spawn several copies and come back after you kill them
  (Overworld enemies spawn several copies ...). Don't set the
  room-clear/kill flags, and spawn each enemy more than once.
- [ ] Hyrule Castle guards chase Link at double speed (Soldiers chase
  Link at double speed / Soldiers move faster). `ovl_En_Heishi*`.
- [ ] Octoroks spit something other than rocks.
  `ovl_En_Okuta`: spawn another actor instead of the rock.
- [ ] Every enemy moves with the speed of the actor before it in the
  actor list (Every enemy moves with the speed of the sprite in the
  previous slot). `Actor_UpdateAll` / `Actor_MoveXZGravity` in
  `src/code/z_actor.c`.
- [ ] Every enemy pushes the actor after it in the actor list instead of
  moving itself (Every enemy pushes the sprite in the next slot ...).
  `src/code/z_actor.c`.

## Items and objects

- [ ] Items are collected anywhere in the room (Items are collected
  anywhere on the screen). Widen the pickup distance check in
  `ovl_En_Item00` (`src/code/z_en_item00.c`).
- [ ] Lift every rock without the Goron's Bracelet or gauntlets (Lift
  every rock without the Power Glove). Skip the strength upgrade check
  when lifting (`ovl_En_Ishi`, `ovl_Obj_Hamishi`, silver/black boulders).
- [ ] Bushes and rocks stay in place after Link lifts them. `ovl_En_Kusa`
  and `ovl_En_Ishi`: don't remove the original.
- [ ] Pots stay in place after Link lifts them. `ovl_Obj_Tsubo`.

## Sound and music

- [ ] No music. Stop the BGM player or mute the sequence channel.
- [ ] Every song plays with the next instrument. Add 1 to the instrument
  number when a sequence sets one (`src/audio/`).
- [ ] Music and sound effects play 2 whole tones higher. Add 4 semitones
  in the note pitch calculation (`src/audio/`).
- [ ] Every sound effect comes from the wrong sound list. Shift the sound
  bank part of each sfx id in `Audio_PlaySfxGeneral`.
