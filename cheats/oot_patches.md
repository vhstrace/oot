# The Legend of Zelda: Ocarina of Time (US NTSC 1.0) patches and GameShark codes

Made in cloud sessions on the `cheat-hunt` branch. See `.claude/CLAUDE.md`
for the rules. Each patch's C change is in `patches/<slug>.diff`. The Mac
builds them into BPS patches. Newest entry last.

## Adult Link rides his horse backwards

**Source diff:** `cheats/patches/adult-link-rides-horse-backwards.diff`
**Changes:** built on the Mac. Changes the `ovl_player_actor` overlay.
**Recorded:** 2026-11-10 23:58:11

**Mechanism:** While Link rides Epona, `Player_Action_8084CC98`
(`ovl_player_actor/z_player.c`) sets his facing to the horse's facing
each frame:

```c
this->yaw = this->actor.shape.rot.y = rideActor->actor.shape.rot.y;
```

The patch keeps `this->yaw` the same as the horse, but turns Link's
model half a turn (`shape.rot.y = horse + 0x8000`). He sits in the saddle
facing the horse's tail. The horse still goes where the stick points.

**Evidence:** `Player_Action_8084CC98` is the riding action (set up by
`Player_SetupActionPreserveAnimMovement(..., Player_Action_8084CC98, 0)`).
The same kind of line runs once when Link gets on
(`Player_ActionHandler_3`), but the riding action overwrites it each
frame.
`EnHorse` reads `player->actor.shape.rot.y` only when Link is not riding
(the horse turns away from him). No `#if` checks in this code.
`git apply --check` passes, and it builds.

**Revisions:** US NTSC 1.0 only. Others not checked.

**Caveats:**

- Not tested.
- Link's riding animation still plays, so his legs and arms point the
  wrong way on the saddle.
- Things that use Link's facing while riding (bow aim, sword swings, the
  camera behind him) may point backwards too.

## Debug object markers are shown

**Source diff:** `cheats/patches/debug-display-objects-shown.diff`
**Changes:** built on the Mac. Changes 20 actor overlays: `ovl_En_Anubice_Tag`,
`ovl_En_Changer`, `ovl_En_Dnt_Demo`, `ovl_En_Encount1`, `ovl_En_Fire_Rock`,
`ovl_En_Fr`, `ovl_En_G_Switch`, `ovl_En_Heishi1`, `ovl_En_Kakasi2`,
`ovl_En_Okarina_Effect`, `ovl_En_Okarina_Tag`, `ovl_En_Reeba`, `ovl_En_Skj`,
`ovl_En_Syateki_Itm`, `ovl_En_Wall_Tubo`, `ovl_En_Weather_Tag`,
`ovl_En_Wonder_Item`, `ovl_En_Wonder_Talk`, `ovl_En_Wonder_Talk2`,
`ovl_En_Yabusame_Mark`.
**Recorded:** 2026-11-10 23:58:11

**Mechanism:** `src/code/z_debug_display.c` (`DebugDisplay_AddObject`,
`DebugDisplay_DrawObjects`) is in the retail `code`, and `Play_Draw`
already draws the list in retail:

```c
if (!DEBUG_FEATURES || (R_HREG_MODE != HREG_MODE_PLAY) || R_PLAY_DRAW_DEBUG_OBJECTS) {
    DebugDisplay_DrawObjects(this);
}
```

But every actor that adds an object does it behind
`if (DEBUG_FEATURES && BREG(0) != 0)` (or `#if DEBUG_FEATURES`), so retail
never adds one. The patch changes each of those gates to `if (1)` (and
`#if 1`). Invisible actors now show a marker: hidden tags for songs and
talking spots (`En_Wonder_Talk`, `En_Okarina_Tag`), item/fairy spawners
(`En_Wonder_Item`), enemy spawners (`En_Encount1`), weather tags, the
Hyrule Castle guards (`En_Heishi1`, a marker over each guard), frogs,
Skull Kid, the silver rupee tracker, archery targets and more. Most are
the I8 circle/cross/ball sprites or the arrow model from
`gameplay_keep`.

**Evidence:** every caller of `DebugDisplay_AddObject` in
`src/overlays/` (grep), 20 files. `DebugDisplay_Init` clears the list
at the start of `Play_Main` each frame. `En_Heishi1` kept no check on
`this->path == BREG(1)`; `En_G_Switch` keeps its check for the silver
rupee tracker type. `git apply --check` passes, and it builds.

**Revisions:** US NTSC 1.0 only. Others not checked.

**Caveats:**

- Not tested.
- The markers are drawn in front of the scene and may cover things. Each
  one costs a little graphics memory per frame; a room with many tags may
  run slower.
- Some markers flash (the actor adds one only every other frame).

## Link runs twice as fast

**Source diff:** `cheats/patches/link-runs-twice-as-fast.diff`
**Changes:** built on the Mac. Changes the `ovl_player_actor` overlay.
**Recorded:** 2026-11-11 00:00:05

**Mechanism:** This is the OoT version of the ALTTP cheat "Link walks
twice as fast". `Player_CalcSpeedAndYawFromControlStick`
(`ovl_player_actor/z_player.c`) turns the stick into a target speed and
caps it at `this->unk_880`:

```c
*outSpeedTarget = (*outSpeedTarget * 0.14f) - (8.0f * floorPitchInfluence * floorPitchInfluence);
*outSpeedTarget = CLAMP(*outSpeedTarget, 0.0f, speedCap);
```

`unk_880` comes from `R_RUN_SPEED_LIMIT / 100.0f` (6.0 with the Kokiri
boots, 5.0 indoors, 3.0 with the Iron Boots, from `sBootData` in
`z_player_lib.c`). The patch changes `0.14f` to `0.28f`, and changes
`R_RUN_SPEED_LIMIT / 100.0f` to `/ 50.0f` where it sets `unk_880`
(three places in `Player_ProcessSceneCollision`) and in the clamp in `func_8083DFE0`. Target speed and cap are
both doubled, so a full stick runs at 12 instead of 6.

**Evidence:** `R_RUN_SPEED_LIMIT` is `REG(45)` (`include/regs.h`),
set per boots by `Player_SetBootData`. Left alone: the
`gSaveContext.entranceSpeed` clamp (speed kept through a door) and
`func_8084AEEC` (`R_RUN_SPEED_LIMIT / 100.0f * 0.8f`, a slide speed
cap). No `#if` checks in the changed lines. `git apply --check`
passes, and it builds.

**Revisions:** US NTSC 1.0 only. Others not checked.

**Caveats:**

- Not tested.
- Acceleration is not changed, so Link takes longer to reach top speed.
- Running animations are not sped up, so Link's feet slide.
- At this speed Link may run off ledges sooner and clip through thin
  walls more easily.
- Swimming, rolling, jumping and riding Epona are not changed.

## Link walks through walls

**Source diff:** `cheats/patches/link-walks-through-walls.diff`
**Changes:** built on the Mac. Changes the `ovl_player_actor` overlay.
**Recorded:** 2026-11-11 00:00:54

**Mechanism:** `Player_ProcessSceneCollision`
(`ovl_player_actor/z_player.c`) picks the background checks for Link,
almost always with `UPDBGCHECKINFO_FLAG_0` (check wall), then calls:

```c
Actor_UpdateBgCheckInfo(play, &this->actor, vWallCheckHeight, vWallCheckRadius, ceilingCheckHeight, flags);
```

With that flag, `Actor_UpdateBgCheckInfo` (`src/code/z_actor.c`) runs
`BgCheck_EntitySphVsWall3` and pushes the actor back out of any wall.
The patch adds `flags &= ~UPDBGCHECKINFO_FLAG_0;` and clears
`BGCHECKFLAG_WALL` just before the call. Floors and ceilings are still
checked, so Link still stands on the ground.

**Evidence:** flag names from `include/actor.h` (`FLAG_0` "check
wall", `FLAG_1` ceiling, `FLAG_2` floor and water). Without
`FLAG_0`, `Actor_UpdateBgCheckInfo` skips the wall block and leaves
`bgCheckFlags` alone, so the patch clears `BGCHECKFLAG_WALL` itself.
This is the same idea as the debug noclip (`sNoclipEnabled`, debug
builds only), but Link keeps gravity. No `#if` checks here.
`git apply --check` passes, and it builds.

**Revisions:** US NTSC 1.0 only. Others not checked.

**Caveats:**

- Not tested.
- The separate "touching a wall" line test (`Player_PosVsWallLineTest`)
  is not changed. Link may still grab ledges, push blocks, and climb
  vines when he walks into them.
- Walking out past the edge of a room can drop Link into the void
  (he respawns with damage), or put him in a spot where a door or
  loading zone never triggers.
- Collision with actors (enemies, NPCs, doors, chests) is a different
  system and still works.

## All floors are slippery like ice

**Source diff:** `cheats/patches/all-floors-slippery-like-ice.diff`
**Changes:** built on the Mac. Changes the `ovl_player_actor` overlay.
**Recorded:** 2026-11-11 00:02:03

**Mechanism:** In `Player_UpdateCommon` (`ovl_player_actor/z_player.c`)
Link keeps his momentum on floor type 5 (the slippery floor type) and
with the Hover Boots:

```c
if (((this->actor.bgCheckFlags & BGCHECKFLAG_GROUND) && (sFloorType == FLOOR_TYPE_5) &&
     (this->currentBoots != PLAYER_BOOTS_IRON)) ||
    ((this->currentBoots == PLAYER_BOOTS_HOVER) && ...)) {
```

Inside, `actor.speed` steps toward the wanted speed by only 0.35 per
frame and `world.rot.y` turns slowly, so Link slides and drifts. The
patch replaces `(sFloorType == FLOOR_TYPE_5)` with `(1)`, so this runs
on any ground. The Iron Boots still stop the sliding, as on real slippery
floors.

**Evidence:** `FLOOR_TYPE_5` is used nowhere else in `src/` (grep).
`sFloorType` is set from `SurfaceType_GetFloorType` in
`Player_ProcessSceneCollision`. The patch is in the player only, not in
`z_bgcheck.c`, so other floor types (lava, quicksand, voids) still work.
No `#if` checks here. `git apply --check` passes, and it builds.

**Revisions:** US NTSC 1.0 only. Others not checked.

**Caveats:**

- Not tested.
- Only Link slides. Enemies, Epona and other actors walk as normal.
- Standing still on a slope may still slide Link down as before; turning
  around fast makes him stop first, then speed up the other way.

## The game runs at half speed

**Source diff:** `cheats/patches/game-runs-at-half-speed.diff`
**Changes:** built on the Mac. Changes `code` (`src/code/graph.c`).
**Recorded:** 2026-11-11 00:03:29

**Mechanism:** `Graph_TaskSet00` (`src/code/graph.c`) tells the
scheduler how many video frames (VIs) each new picture must stay on
screen:

```c
cfb->updateRate = R_UPDATE_RATE;
```

`R_UPDATE_RATE` is 3 during play (20 frames per second on NTSC), and 1
on the title screen and the file select. The scheduler
(`Sched_HandleRetrace` in `src/code/sched.c`) won't swap to the next
picture until `updateTimer` counts down, and the game can't start a
new frame until a frame buffer is free. The patch changes the line to
`R_UPDATE_RATE * 2`, so play runs at 10 frames per second, half speed.

**Evidence:** `R_UPDATE_RATE` is `SREG(30)` (`include/regs.h`), set
to 3 in `GameState_Init` (`game.c`) and by `Play_Update` after
transitions. `updateRate` is an `s8` in `CfbInfo`
(`include/sched.h`). `R_UPDATE_RATE` itself is not changed, so code
that scales motion by it (`Actor_UpdatePos` uses
`R_UPDATE_RATE * 0.5f`) moves the same distance per frame as before.
`git apply --check` passes, and it builds.

**Revisions:** US NTSC 1.0 only. Others not checked.

**Caveats:**

- Not tested.
- Music and sound effects run on their own timer and stay at normal
  speed. Cutscenes timed to music may drift from it.
- The title screen and menus run at 30 frames per second instead of 60.

## The game runs 1.5 times as fast

**Source diff:** `cheats/patches/game-runs-faster.diff`
**Changes:** built on the Mac. Changes `code` (`src/code/graph.c`).
**Recorded:** 2026-11-11 00:03:31

**Mechanism:** Same line as "The game runs at half speed". During play
`R_UPDATE_RATE` is 3, so each picture stays up for 3 video frames (20
frames per second on NTSC). The patch changes

```c
cfb->updateRate = R_UPDATE_RATE;
```

to `R_UPDATE_RATE - 1`. Play now waits only 2 video frames per picture
(30 frames per second), so everything happens up to 1.5 times as fast.
On the title screen and menus (rate 1) it becomes 0, which the
scheduler treats the same as 1 (it swaps when `updateTimer <= 0`).

**Evidence:** see "The game runs at half speed". The game logic counts
frames, not time, so timers, enemy moves and Link all speed up together.
`R_UPDATE_RATE` itself is not changed, so movement per frame stays the
same and collision is no less precise. Going to 1 (60 frames per second,
3 times as fast) was left out: the N64 CPU can't run OoT that fast, so
it would only make the speed jump around. `git apply --check` passes,
and it builds.

**Revisions:** US NTSC 1.0 only. Others not checked.

**Caveats:**

- Not tested.
- The real gain depends on how busy the scene is. OoT already drops
  below 20 frames per second in big areas (Hyrule Field, Kakariko); those
  get little or no faster on real hardware or an accurate emulator.
  An emulator with CPU overclock helps.
- Music and sound effects stay at normal speed, so cutscenes timed to
  music may drift from it.
- Timed challenges (Gerudo archery, the Dampé race) also count frames, so
  they get no harder or easier.

## Frame advance on controller 1 (L pauses, R steps one frame)

**Source diff:** `cheats/patches/frame-advance-on-controller-1.diff`
**Changes:** built on the Mac. Changes `code` (`z_frame_advance.c`,
`z_play.c`).
**Recorded:** 2026-11-11 00:05:19

**Mechanism:** The debug frame advance is still in the retail game.
`Play_Update` (`src/code/z_play.c`) runs the whole game update only
when `FrameAdvance_Update` returns true:

```c
if (FrameAdvance_Update(&this->frameAdvCtx, &input[1])) {
```

`FrameAdvance_Update` (`src/code/z_frame_advance.c`) reads controller 2:
R + D-Down turns it on or off, and Z + R steps one frame (holding Z + R
steps every 9 frames). The patch:

- passes `&input[0]` (controller 1) instead, but keeps `&input[1]` while
  the pause menu is open (`IS_PAUSED(&this->pauseCtx)`), so L in the
  menu doesn't freeze it;
- turns frame advance on or off with a press of L;
- steps one frame on a press of R, and keeps stepping every 9 frames
  while R is held (no Z needed).

**Evidence:** the call in `Play_Update` is outside every `#if
DEBUG_FEATURES` block, so retail has it. `FrameAdvance_IsEnabled` (used
by `z_effect.c`, `z_effect_soft_sprite.c`, `z_kankyo.c`,
`z_scene_table.c` and the collision debug draw) also stops effects and
scrolling textures while paused. `Play_Draw` still runs, so the frozen
scene stays on screen. `git apply --check` passes, and it builds.

**Revisions:** US NTSC 1.0 only. Others not checked.

**Caveats:**

- Not tested.
- R is still the shield button, so Link raises his shield on each frame
  you step.
- L also toggles the minimap (`z_map_exp.c`). The press that turns
  frame advance off runs a frame with L down, so the minimap may toggle.
- Music keeps playing while the game is frozen.

## Time moves only while a button or the stick is pressed

**Source diff:** `cheats/patches/time-moves-only-while-input-held.diff`
**Changes:** built on the Mac. Changes `code` (`z_frame_advance.c`,
`z_play.c`).
**Recorded:** 2026-11-11 00:05:21

**Mechanism:** This is the OoT version of the ALTTP cheat "Time moves
only while R is held". It uses the retail game's frame advance (see
"Frame advance on controller 1"). `Play_Update` runs the game update only
when `FrameAdvance_Update` returns true. The patch replaces the body of
`FrameAdvance_Update` with:

```c
frameAdvCtx->enabled = !((input->cur.button != 0) || (input->cur.stick_x >= 10) || (input->cur.stick_x <= -10) ||
                         (input->cur.stick_y >= 10) || (input->cur.stick_y <= -10));
return !frameAdvCtx->enabled;
```

and `Play_Update` passes controller 1 (`&input[0]`) and always runs
while the pause menu is open:

```c
if (FrameAdvance_Update(&this->frameAdvCtx, &input[0]) || IS_PAUSED(&this->pauseCtx)) {
```

So the game runs only on frames where some button is down or the stick
is pushed past 10 (out of about 80). Setting `enabled` also freezes the
effects and texture scrolling that check `FrameAdvance_IsEnabled`.

**Evidence:** `Input` is in `include/libu64/pad.h`; `cur` is the raw
`OSContPad` (`button`, `stick_x`, `stick_y`). A press on a frozen
frame is still seen, since that frame then runs with the button down.
`git apply --check` passes, and it builds.

**Revisions:** US NTSC 1.0 only. Others not checked.

**Caveats:**

- Not tested.
- Everything stops without input: text boxes, cutscenes, scene changes
  and the Game Over screen. Hold any button (C-Up, D-pad or L are safe in
  most places) to let them run.
- Music keeps playing while time is stopped.
- Some worn N64 sticks rest past 10 and keep time moving; raise the 10 if
  so.

## Every chest opens with the short kick

**Source diff:** `cheats/patches/chests-open-with-no-delay.diff`
**Changes:** built on the Mac. Changes the `ovl_player_actor` overlay.
**Recorded:** 2026-11-11 00:06:20

**Mechanism:** This is the OoT version of the ALTTP cheat "No delay for
opening treasure chests". When Link opens a chest,
`Player_ActionHandler_2` (`ovl_player_actor/z_player.c`) picks one of
two ways:

```c
if ((giEntry->itemId != ITEM_NONE) && (giEntry->gi >= 0) &&
    (Item_CheckObtainability(giEntry->itemId) == ITEM_NONE)) {
    Player_AnimPlayOnceAdjusted(play, this, this->ageProperties->unk_98);
    ...
    chest->unk_1F4 = 1;
    Camera_RequestSetting(Play_GetCamera(play, CAM_ID_MAIN), CAM_SET_SLOW_CHEST_CS);
} else {
    Player_AnimPlayOnce(play, this, &gPlayerAnim_link_normal_box_kick);
    chest->unk_1F4 = -1;
}
```

The first is the long cutscene for new major items (`link_demo_Tbox_open`,
slow camera, the light beam `Demo_Tre_Lgt` and the chest fanfare, started
by `EnBox_WaitOpen` when `unk_1F4 > 0`). The second is the quick kick
used for small items. The patch adds `0 &&` to the condition, so every
chest uses the quick kick.

**Evidence:** `GET_ITEM` (top of `z_player.c`) makes `gi` negative for
`CHEST_ANIM_SHORT`. `EnBox_WaitOpen` (`ovl_En_Box/z_en_box.c`) spawns
`ACTOR_DEMO_TRE_LGT` and plays `NA_BGM_OPEN_TRE_BOX` only when
`unk_1F4 > 0`; with `-1` it plays the short lid animation. The item
is still given, with its normal get-item text, by
`Player_Action_8084E6D4`. `git apply --check` passes, and it builds.

**Revisions:** US NTSC 1.0 only. Others not checked.

**Caveats:**

- Not tested.
- Big items lose their chest light and fanfare. The get-item pose,
  jingle and text box are not changed, so some wait is still there.
- Link holds the item up with the small-item pose (`get_itemB`).

## Link picks up pots, bushes and rocks 3 times as fast

**Source diff:** `cheats/patches/link-lifts-with-almost-no-delay.diff`
**Changes:** built on the Mac. Changes the `ovl_player_actor` overlay.
**Recorded:** 2026-11-11 00:07:25

**Mechanism:** `func_8083A0F4` (`ovl_player_actor/z_player.c`) starts
every lift: it picks the action and animation (`carryB` for pots, bushes
and small rocks, `link_silver_carry` for silver rocks,
`link_normal_heavy_carry` for the big black boulders,
`link_normal_nocarry_free` for trying to lift without strength), then
plays it at normal speed:

```c
Player_AnimPlayOnce(play, this, anim);
```

The patch changes that line to
`LinkAnimation_PlayOnceSetSpeed(play, &this->skelAnime, anim, 3.0f);`.
The lift actions (`Player_Action_80846050`, `_80846260`,
`_80846120`, `_80846408`) wait for the animation to end and grab the
object on a set frame, so the whole lift takes about a third of the time.

**Evidence:** `LinkAnimation_OnFrame` (`src/code/z_skelanime.c`)
checks whether the frame was crossed since last update, using
`playSpeed`, so the grab frames (4, 27, 41, 229) still fire at 3x speed.
The heavy boulder still waits 20 frames after its animation
(`this->av2.actionVar2++ > 20`). `git apply --check` passes, and it
builds.

**Revisions:** US NTSC 1.0 only. Others not checked.

**Caveats:**

- Not tested.
- Throwing and putting things down are not changed.
- The animation blends in from the previous pose over a few frames
  (the morph), so it isn't instant.

## A new game starts with the debug save

**Source diff:** `cheats/patches/new-game-starts-with-debug-save.diff`
**Changes:** built on the Mac. Changes `code` (`src/code/z_sram.c`).
**Recorded:** 2026-11-11 00:11:13

**Mechanism:** This is the OoT version of the ALTTP cheat "Start a new
game with BAGE debug loadout". `Sram_InitSave` (`src/code/z_sram.c`)
makes a new file. Debug builds give file 1 the debug save; retail always
makes a blank one:

```c
#if DEBUG_FEATURES
    if (fileSelect->buttonIndex != 0) {
        Sram_InitNewSave();
    } else {
        Sram_InitDebugSave();
    }
#else
    Sram_InitNewSave();
#endif
    ...
    gSaveContext.save.cutsceneIndex = CS_INDEX_1;
```

The patch changes the retail `Sram_InitNewSave()` to
`Sram_InitDebugSave()`, and `CS_INDEX_1` to `CS_INDEX_NONE` (debug
builds also skip the intro for file 1). Every new file now starts in
Link's house as child Link with the debug save: 14 hearts, magic, 150
rupees, all three swords, shields and tunics, Bow, Fire/Ice/Light Arrows,
Din's Fire, Farore's Wind, Nayru's Love, Slingshot, Fairy Ocarina,
Bombchus, Hookshot, Boomerang, Lens of Truth, Magic Bean, Megaton Hammer,
four bottles (empty, red, green, blue potions), the Pocket Egg and the
Weird Egg.

**Evidence:** `Sram_InitDebugSave` and its data (`sDebugSavePlayerData`,
`sDebugSaveEquips`, `sDebugSaveInventory`) are in the retail `code`;
the title-screen demo (`ovl_opening/z_opening.c`) and the map select
call it. It sets some early story flags (Saria greeted Link, Mido, Zelda
fled the castle). `Sram_InitSave` sets the entrance, age and time after
it, and copies the file name over the debug name "LINK".
`git apply --check` passes, and it builds.

**Revisions:** US NTSC 1.0 only. Others not checked.

**Caveats:**

- Not tested.
- `magicLevel` is set to 0 by the debug save even though magic is
  "acquired", so the magic meter may not show until a magic jar or a
  Great Fairy sets it.
- Child Link can't use the adult items (Bow, Hookshot, Hammer) until he
  grows up; they show grayed out.
- No intro cutscene, so Navi's first meeting is skipped. Story flags for
  later events are not set, so the story still runs from the start.
- Files already saved are not changed.

## Loading File 1 opens the map select

**Source diff:** `patches/file-1-opens-map-select.diff`
**Changes:** built on the Mac. Changes the `ovl_file_choose` overlay.
**Recorded:** 2026-11-11 00:13:00

**Mechanism:** `FileSelect_LoadGame` in `z_file_choose.c` has a block
that only the debug ROM builds (`#if DEBUG_FEATURES`). When the player
loads File 1, it opens the save and then goes to `MapSelect_Init`
instead of `Play_Init`. The patch changes `#if DEBUG_FEATURES` to
`#if 1`, so the retail build has the block too.

**Evidence:** `ovl_select` (the map select) and `z_select_static` are
already in the retail ROM (`ovl_select.plf` is in the map), so only this
one gate is missing. `git apply --check` passes, and it builds.

**Revisions:** US NTSC 1.0 only. Others not checked.

**Caveats:**

- Not tested.
- File 1 must exist. Make a file in slot 1 first, then load it.
- The map select uses File 1's save, not the debug save, since
  `gSaveContext.fileNum` is 0 and not `0xFF`.
- Files 2 and 3 load the game as before.
- The other controller 2 debug features (L+R+Z to the map select and so on)
  stay off. They are `#if DEBUG_FEATURES` code in `graph.c` and other
  files.

## A new game starts with the Kokiri Sword and Deku Shield

**Source diff:** `patches/new-game-starts-with-sword-and-shield.diff`
**Changes:** built on the Mac. Changes `code` (`z_sram.c`).
**Recorded:** 2026-11-11 00:13:46

**Mechanism:** `Sram_InitNewSave` copies `sNewSaveEquips` and
`sNewSaveInventory` into a new save. In the original, Link owns and wears
only the Kokiri Tunic and Kokiri Boots (`equipment` `0x1100`), and the B
button is empty. The patch:

- puts `ITEM_SWORD_KOKIRI` on the B button;
- changes the worn `equipment` to `0x1111` (Kokiri Sword and Deku
  Shield worn, plus the tunic and boots);
- adds `EQUIP_INV_SWORD_KOKIRI` and `EQUIP_INV_SHIELD_DEKU` to the owned
  equipment.

The TODO idea was "Link's uncle gives the Master Sword" (from Zelda 3).
Child Link can't hold the Master Sword, so this gives the child sword and
shield.

**Evidence:** the equipment nibbles are sword (bits 0-3), shield (4-7),
tunic (8-11) and boots (12-15), and `EQUIP_VALUE_SWORD_KOKIRI` and
`EQUIP_VALUE_SHIELD_DEKU` are both 1. The debug save does the same for the
sword (`buttonItems[0] = ITEM_SWORD_KOKIRI`). The save checksum is made
again when the file is written, so the constant `sNewSaveChecksum` does
not need a change. `git apply --check` passes, and it builds.

**Revisions:** US NTSC 1.0 only. Others not checked.

**Caveats:**

- Not tested.
- Only new files get this. Old saves are not changed.
- Mido checks for the sword and shield, so he should let Link pass to the
  Deku Tree at once.
- The Kokiri Sword chest and the Deku Shield in the shop are still there.
  Getting them again does nothing new.

## Link's house door leads to the Chamber of the Sages

**Source diff:** `patches/links-house-door-leads-to-sages-chamber.diff`
**Changes:** built on the Mac. Changes the `ovl_player_actor` overlay.
**Recorded:** 2026-11-11 00:15:02

**Mechanism:** when Link walks onto an exit floor,
`Player_HandleExitsAndVoids` in `z_player.c` does
`play->nextEntranceIndex = play->exitList[exitIndex - 1];`. The patch adds
a check right after it: if the scene is `SCENE_LINKS_HOUSE`, the next
entrance becomes `ENTR_CHAMBER_OF_THE_SAGES_0` (`0x06B`). This is the
OoT version of the Zelda 3 "Link's house door leads to the Chris Houlihan
room" idea.

**Evidence:** Link's house has one doorway, and both entries of its exit
list are `ENTR_KOKIRI_FOREST_3`
(`link_home_scene_020000A0_ExitList`). The test maps (`testroom`,
`test01`, `syotes`, `sutaru` and others) are only in the GameCube and
iQue specs (`spec/scenes_gc_ique.inc`), not in `spec/scenes_n64.inc`, so
the N64 ROM has no test map to send Link to. The Chamber of the Sages is
in the ROM, but the player only sees it in cutscenes. `git apply --check`
passes, and it builds.

**Revisions:** US NTSC 1.0 only. Others not checked.

**Caveats:**

- Not tested.
- The Chamber of the Sages has no exit. To leave, use a warp song, Farore's
  Wind, or save and reload. Before Link has any of these, save and reload
  is the only way out.
- What shows up in the chamber (Rauru, sages, a cutscene) depends on the
  scene's setup for the game's progress. A cutscene might play, or Link
  might be stuck in a state meant for one.
- Only the exit from Link's house changes. Going into the house works as
  before.

## Every enemy dies in one hit

**Source diff:** `patches/every-enemy-dies-in-one-hit.diff`
**Changes:** built on the Mac. Changes `code` (`z_collision_check.c`).
**Recorded:** 2026-11-11 00:15:55

**Mechanism:** `CollisionCheck_ApplyDamage` works out the damage of each
hit (from the actor's damage table, or attack minus defense) and adds it
with `col->actor->colChkInfo.damage += damage;`. The actor then takes it
off its health, most often with `Actor_ApplyDamage`. The patch adds a
check after the add: if the hit actor is `ACTORCAT_ENEMY` and the damage
is not 0 but less than its health, the damage becomes its health. So any
hit that does damage at all takes the enemy's health to 0.

**Evidence:** `Actor_ApplyDamage` sets health to 0 when
`health <= damage`. Setting the damage to exactly the health (and not
`0xFF`) also works for actors that do `health -= damage` by themselves,
since the u8 can't wrap. `git apply --check` passes, and it builds.

**Revisions:** US NTSC 1.0 only. Others not checked.

**Caveats:**

- Not tested.
- Only actors in the enemy category. Bosses (`ACTORCAT_BOSS`) take normal
  damage.
- Hits with 0 damage stay at 0. Deku Nuts, the boomerang and other attacks
  that only stun still only stun, and enemies that are immune to an attack
  stay immune.
- Some enemies keep their own health count or need a special attack to
  finish them (for example, enemies that must be stunned first). They may
  not die at once.
- Enemy-on-enemy hits (rare) are also deadly.

## The spin attack charges at once

**Source diff:** `patches/spin-attack-charges-at-once.diff`
**Changes:** built on the Mac. Changes the `ovl_player_actor` overlay.
**Recorded:** 2026-11-11 00:16:47

**Mechanism:** `this->unk_858` is the spin attack charge (0.0 to 1.0).
When Link starts charging by holding B, `func_80837704` calls
`func_80837530(play, this, 0x200)`, which sets
`this->unk_858 = 0.0f;`. Then `func_80844E3C` adds 0.02 each frame
(`Math_StepToF(&this->unk_858, 1.0f, 0.02f)`), so a full charge takes 50
frames. The patch starts the charge at `1.0f` instead.

**Evidence:** on release, `func_80844BE4` picks the big spin when
`this->unk_858 >= 0.85f`. `EnMThunder` (the spin effect) reads the same
value for its glow and rumble. `git apply --check` passes, and it builds.

**Revisions:** US NTSC 1.0 only. Others not checked.

**Caveats:**

- Not tested.
- Holding B after a swing and letting go always gives the big charged spin.
  The charge-up sound and the glow start at full.
- The quick spin from rolling the stick (`func_80837530(play, this, 0)`)
  still starts at 0.5, as before.
- With the Great Spin magic upgrade, every charged spin uses magic for the
  big spin, as a full charge does.

## Hold B to keep swinging the sword

**Source diff:** `patches/hold-b-to-keep-swinging.diff`
**Changes:** built on the Mac. Changes the `ovl_player_actor` overlay.
**Recorded:** 2026-11-11 00:17:40

**Mechanism:** `func_8083BB20` in `z_player.c` decides if Link starts a
sword swing. For the player it returns true only when `sUseHeldItem` is
set, which happens on a new press of the button. `Player_ActionHandler_7`
calls it to start a swing, and `Player_Action_808502D0` calls
`Player_ActionHandler_7` when a swing ends, to chain the next one. The
patch changes `if (sUseHeldItem ||` to
`if (sUseHeldItem || sHeldItemButtonIsHeldDown ||`.
`sHeldItemButtonIsHeldDown` is true each frame the button for the held
item is down (set in `Player_ProcessItemButtons`). So with B held, a new
swing starts as soon as the last one ends.

**Evidence:** `sHeldItemButtonIsHeldDown` is cleared each frame and set
again only while the button of the item in Link's hand is held.
`func_8083BB20` also needs a melee weapon in hand and no shield up.
`git apply --check` passes, and it builds.

**Revisions:** US NTSC 1.0 only. Others not checked.

**Caveats:**

- Not tested.
- Each swing still plays to its end. Only the wait for a new press is gone.
- Holding B now swings again in place of charging, so the charged spin
  attack can't start. The quick spin (roll the stick, then B) still works.
- Holding a C button with a Deku Stick in hand also swings again and again.

## The sword hits enemies across most of the screen

**Source diff:** `patches/sword-hits-across-the-screen.diff`
**Changes:** built on the Mac. Changes `code` (`z_player_lib.c`).
**Recorded:** 2026-11-11 00:18:36

**Mechanism:** the sword's two hit boxes are `meleeWeaponQuads[0]` and
`[1]`. Each frame of a swing, `Player_CalcMeleeWeaponTipPositions`
works out their tips from `sMeleeWeaponTipOffsetFromLeftHand1` and
`2` (in the hand's model space). Their length is the sword length from
`sMeleeWeaponLengths` (3000 for the Kokiri Sword, 4000 for the Master
Sword) plus 1200: `sMeleeWeaponTipOffsetFromLeftHand1.x += 1200.0f;`. The
patch adds `30000.0f` instead. At Link's scale (0.01) the hit boxes
reach about 300 units farther, so a swing sweeps a wide arc around Link.

**Evidence:** `Player_UpdateMeleeWeaponInfo` uses only tip positions 1
and 2 for the two quads. Tip 0 (`sMeleeWeaponTipOffsetFromLeftHand0`) is
for the sword trail and the wall checks in `z_player.c`
(`meleeWeaponInfo[0]`), and it doesn't change. `git apply --check`
passes, and it builds.

**Revisions:** US NTSC 1.0 only. Others not checked.

**Caveats:**

- Not tested.
- The sword model and its trail look the same. Only the hit boxes grow.
- The hit boxes go through walls, so the sword can hit enemies, switches,
  pots and grass behind walls.
- It also works for the Hammer, Biggoron's Sword and the Deku Stick. The
  Deku Stick sets its own length (`this->unk_85C * 5000.0f`), but it goes
  through the same function, so it gets the long reach too.
- The hit boxes are long but still thin. An enemy right above or below the
  swing's plane can still be missed.

## The shield blocks hits from any direction

**Source diff:** `patches/shield-blocks-from-any-direction.diff`
**Changes:** built on the Mac. Changes the `ovl_player_actor` overlay.
**Recorded:** 2026-11-11 00:19:52

**Mechanism:** Link's shield is a collider (`shieldQuad`) in front of him.
A hit that touches it sets `AC_BOUNCED`, and `func_808382DC` (the player's
damage handling) treats that as a block:
`sp64 = (this->shieldQuad.base.acFlags & AC_BOUNCED) != 0;` and
`if (sp64 || ...) { ... return 0; }`. A hit from the side or back touches
Link's body collider (`cylinder`) instead and does damage. The patch adds
one more case to that `if`: Link is shielding
(`PLAYER_STATE1_SHIELDING`), has a shield (`currentShield !=
PLAYER_SHIELD_NONE`), and his body was hit (`cylinder.base.acFlags &
AC_HIT`). Then the hit is blocked the same way, with the shield-hit
recoil and no damage.

**Evidence:** the new case is its own condition, so `sp64` doesn't change
and the fire check after it (which reads
`shieldQuad.elem.acHitElem`) still runs only for real shield hits. The
blocking branch returns before `func_80837C0C`, which is what takes the
health off (`func_80837B18(play, this, 0 - this->actor.colChkInfo.damage)`).
`git apply --check` passes, and it builds.

**Revisions:** US NTSC 1.0 only. Others not checked.

**Caveats:**

- Not tested.
- Only while Link holds up a shield (R, or Z-target with a one-handed
  weapon). Without a shield, or with his guard down, hits work as before.
- A shot from behind is not reflected or bounced back like a front shield
  hit. It stops, as if it hit Link, but Link takes no damage.
- Hits with knockback (`knockbackType`, from big attacks, explosions and
  some enemies) are handled before this check and still hurt.
- Child Link with the Hylian Shield blocks without the recoil, as before.

## Thrown pots and crates hit enemies anywhere nearby

**Source diff:** `patches/thrown-pots-hit-anywhere.diff`
**Changes:** built on the Mac. Changes the `ovl_Obj_Tsubo` and
`ovl_Obj_Kibako` overlays.
**Recorded:** 2026-11-11 00:20:54

**Mechanism:** a thrown pot (`ObjTsubo_Thrown`) or small crate
(`ObjKibako_Thrown`) attacks with its cylinder collider
(`AT_ON | AT_TYPE_PLAYER`). It breaks when it touches the ground, a wall,
or something it hits (`AT_HIT`). The cylinder is small: radius 9, height
26 for the pot, and radius 12, height 27 for the crate. In the thrown
state, the patch makes the cylinder radius 300 and height 300, with a
`yShift` of -150 so it reaches 150 units above and below the object, before
`CollisionCheck_SetAT`. So the pot or crate hits the first enemy within
300 units. The patch also drops the `CollisionCheck_SetOC` call in the
thrown state, so the big cylinder doesn't push actors out of the way.

**Evidence:** `Collider_UpdateCylinder` only copies the actor's position
into the collider, so the new size stays set. The object is killed when it
breaks, so the size never goes back to the idle or held state. Rocks
(`ovl_En_Ishi`) have no attack collider (`ATELEM_NONE`) and `EnIshi_Fly`
never calls `CollisionCheck_SetAT`, so they can't hit enemies at all, and
this patch doesn't change them. `git apply --check` passes, and it
builds.

**Revisions:** US NTSC 1.0 only. Others not checked.

**Caveats:**

- Not tested.
- The pot breaks on the first thing in range, often right after the throw.
- Big crates (`ovl_Obj_Kibako2`) and rocks are not changed. Bombs are not
  changed either: a thrown bomb hits with its explosion, not while it
  flies, and a bigger explosion would also hurt Link from far away.
- In flight, the pot or crate is no longer pushed by other actors.

## Link and his horse both face backwards, and the horse runs backwards

**Source diff:** `patches/link-and-horse-face-backwards.diff`
**Changes:** built on the Mac. Changes the `ovl_En_Horse` and
`ovl_player_actor` overlays.
**Recorded:** 2026-11-11 00:23:05

**Mechanism:** this builds on "Adult Link rides his horse backwards". It
has the same `z_player.c` change: in `Player_Action_8084CC98`,
`this->yaw = this->actor.shape.rot.y = rideActor->actor.shape.rot.y;`
becomes `this->yaw = rideActor->actor.shape.rot.y;` and
`this->actor.shape.rot.y = rideActor->actor.shape.rot.y + 0x8000;`, so
Link faces the back of the horse. It also turns the horse model around: in
`EnHorse_Draw`, the patch adds `0x8000` to `shape.rot.y` before
`func_800A6360` (the skin draw) and takes it off again after. The horse
moves and steers as before, but its model points the other way, so it runs
tail first. Link now faces the same way as the drawn horse, so both face
away from where they are going.

**Evidence:** `EnHorse` uses `shape.rot.y` in about 20 places for
movement and steering, so the patch only changes it while the model is
drawn and puts it back right after. The rider position comes from limb 30
of the drawn skin (`Skin_GetLimbPos(skin, 30, &riderOffset,
&this->riderPos)` in `EnHorse_PostDraw`), so Link stays in the saddle of
the turned model. `git apply --check` passes, and it builds.

**Revisions:** US NTSC 1.0 only. Others not checked.

**Caveats:**

- Not tested.
- Don't use it with "Adult Link rides his horse backwards". Both diffs
  change the same line of `z_player.c`, so they don't apply together.
  This diff already has that change.
- Every `EnHorse` is turned around, also when no one rides it: Epona in
  Lon Lon Ranch, Ingo's horse in the race, and horses in cutscenes that
  use this actor.
- The horse's hit spheres (`colliderJntSph`) and the hoof dust are set
  from the drawn skin in `EnHorse_PostDraw`, so they turn around with the
  model. Jumps and wall checks still use the direction of travel.
- The control stick still steers by the direction of travel, as before.

## Arrows fly through enemies

**Source diff:** `patches/arrows-fly-through-enemies.diff`
**Changes:** built on the Mac. Changes the `ovl_En_Arrow` overlay.
**Recorded:** 2026-11-11 00:32:24

**Mechanism:** in `EnArrow_Fly` (`z_en_arrow.c`), `atTouched` is set when
the arrow's collider hit an actor (`this->collider.base.atFlags & AT_HIT`).
Then the arrow stops: it sticks in the actor, carries it (for actors that
can ride an arrow), or bounces off (`func_809B3CEC`). The patch adds a
check before that: for arrows (`params < ARROW_SEED`), it clears `AT_HIT`
and sets `atTouched` to false. So the arrow keeps flying. The hit still
counts for the actor that was hit, since the collision system already gave
it the damage. This is the OoT version of the ALTTP "Sword beams fly
through enemies" cheat.

**Evidence:** the actor's damage is set in `CollisionCheck_ApplyDamage`
before the arrow's update reads `AT_HIT`, so clearing the flag only
changes what the arrow does. The arrow's collider stays on, so it can hit
more enemies on its way. Walls (`this->touchedPoly`) still stop it.
`git apply --check` passes, and it builds.

**Revisions:** US NTSC 1.0 only. Others not checked.

**Caveats:**

- Not tested.
- An arrow can hit the same enemy on more than one frame while it passes
  through. Most enemies are safe for a short time after a hit, so this
  may not matter.
- Fire, ice and light arrows burst into their effect when the arrow
  hits something (`hitFlags`). They no longer burst on enemies, only on
  walls and the floor, or when the arrow's time runs out.
- Enemy arrows (negative `params`) also fly through what they hit,
  including Link.
- Slingshot seeds and Deku Nuts are not changed.

## Arrows fly twice as fast

**Source diff:** `patches/arrows-fly-twice-as-fast.diff`
**Changes:** built on the Mac. Changes the `ovl_En_Arrow` overlay.
**Recorded:** 2026-11-11 00:32:26

**Mechanism:** `EnArrow_Shoot` (`z_en_arrow.c`) starts an arrow with
`Actor_SetProjectileSpeed(&this->actor, 150.0f);` and `this->timer = 12;`.
The patch changes the speed to `300.0f`. The timer stays at 12 frames, so
the arrow also goes twice as far before it disappears. This is the OoT
version of the ALTTP "Sword beams fly twice as fast" cheat.

**Evidence:** the other branch (`params >= ARROW_SEED`, 80.0) is for
slingshot seeds and Deku Nuts, and it doesn't change. `git apply --check`
passes, and it builds.

**Revisions:** US NTSC 1.0 only. Others not checked.

**Caveats:**

- Not tested.
- Arrows reach about 3600 units in place of 1800 before they vanish.
- Gravity starts in the last frames (`this->timer < 7.2`), as before, so
  long shots drop at the end the same way.
- Enemy arrows (negative `params`) also fly twice as fast.
- At this speed, an arrow can move past a thin target in one frame. The
  collider covers the whole path of each frame
  (`Player_UpdateWeaponInfo`), so this should be rare.

## Arrows fly at quarter speed

**Source diff:** `patches/arrows-fly-at-quarter-speed.diff`
**Changes:** built on the Mac. Changes the `ovl_En_Arrow` overlay.
**Recorded:** 2026-11-11 00:32:28

**Mechanism:** `EnArrow_Shoot` (`z_en_arrow.c`) starts an arrow with
`Actor_SetProjectileSpeed(&this->actor, 150.0f);` and `this->timer = 12;`.
The patch changes the speed to `37.5f` and the timer to `48`. So arrows
fly at a quarter of the speed, and still go the same distance (1800 units)
before they vanish. This is the OoT version of the ALTTP "Sword beams fly
at quarter speed" cheat.

**Evidence:** `EnArrow_Fly` kills the arrow when the timer reaches 0, and
turns on gravity (`-0.4`) when `this->timer < 7.2`. Slingshot seeds and
Deku Nuts use the other branch and don't change. `git apply --check`
passes, and it builds.

**Revisions:** US NTSC 1.0 only. Others not checked.

**Caveats:**

- Not tested.
- Enemy arrows (negative `params`) are also slow.
- Moving targets are much harder to hit.
