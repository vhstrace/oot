# The Legend of Zelda: Ocarina of Time (US NTSC 1.0) patches and GameShark codes

Made in cloud sessions on the `cheat-hunt` branch. See `.claude/CLAUDE.md`
for the rules. Each patch's C change is in `patches/<slug>.diff`. The Mac
builds them into BPS patches. Newest entry last.

## Adult Link rides his horse backwards

**Source diff:** `cheats/patches/adult-link-rides-horse-backwards.diff`
**Changes:** built on the Mac. Changes the `ovl_player_actor` overlay.
**Recorded:** 2026-11-10 23:58:11

Tags: `demo-ready` `video-ready`

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

- Tested by the user on 2026-11-11 (RetroArch, BPS patch): works.
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

Tags: `demo-ready` `play-start`

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

- Tested by the user on 2026-11-11 (RetroArch, BPS patch): works. The
  first BPS also had the horse-backwards change by mistake; it was
  rebuilt without it the same day.
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

## Link, Ganondorf and their horses face backwards, and the horses run backwards

**Source diff:** `patches/link-and-horse-face-backwards.diff`
**Changes:** built on the Mac. Changes the `ovl_En_Horse`, `ovl_En_Viewer`
and `ovl_player_actor` overlays.
**Recorded:** 2026-11-11 00:23:05

Tags: `demo-ready` `video-ready`

**Mechanism:** this builds on "Adult Link rides his horse backwards". It
has the same `z_player.c` change: in `Player_Action_8084CC98`,
`this->yaw = this->actor.shape.rot.y = rideActor->actor.shape.rot.y;`
becomes `this->yaw = rideActor->actor.shape.rot.y;` and
`this->actor.shape.rot.y = rideActor->actor.shape.rot.y + 0x8000;`, so
Link faces the back of the horse. It also turns the horse model around: in
`EnHorse_Draw`, the patch adds `0x8000` to `shape.rot.y` before
`func_800A6360` (the skin draw) and takes it off again after. It also
negates `shape.rot.x` (the slope tilt) for the draw and puts it back. The
skin matrix applies the X turn after the Y turn
(`SkinMatrix_SetTranslateRotateYXZScale` in `z_skin_awb.c`), so without
this the turned horse tilts the wrong way on a slope: running uphill, it
stood on its hind legs. The horse
moves and steers as before, but its model points the other way, so it runs
tail first. Link now faces the same way as the drawn horse, so both face
away from where they are going.

Ganondorf and his horse in cutscenes are `En_Viewer` actors: types 3
and 4 (Ganondorf and his rearing horse in the Hyrule Field drawbridge
cutscene) and types 5 and 6 (the opening dream). In `EnViewer_Draw`, for
types 3 to 6, the patch adds `0x8000` to `shape.rot.y` around the draw
call and also calls `Matrix_RotateY(M_PI, MTXMODE_APPLY)`. Ganondorf's
skeleton is drawn with the matrix that `Actor_Draw` already built, so
the extra turn on the matrix turns him. The horse's skin draw builds its
own matrix from `shape.rot`, so the `shape.rot.y` change turns the horse.
Ganondorf's cape (`EnViewer_UpdateGanondorfCape`) also reads
`shape.rot.y`, so it follows him.

**Evidence:** `EnHorse` uses `shape.rot.y` in about 20 places for
movement and steering, so the patch only changes it while the model is
drawn and puts it back right after. The rider position comes from limb 30
of the drawn skin (`Skin_GetLimbPos(skin, 30, &riderOffset,
&this->riderPos)` in `EnHorse_PostDraw`), so Link stays in the saddle of
the turned model. In `EnViewer_UpdatePosition`, Ganondorf and his horse
both take their position from the same cutscene cue (`actorCues[1]`), so
turning each one around that point keeps him in the saddle. The scene
files use types 3 and 4 in `spot00` and types 5 and 6 in `hiral_demo`.
`Skin_DrawImpl` loads its own matrix and does not use the matrix stack,
so the extra matrix turn doesn't turn the horse twice.
`ovl_En_Horse_Ganon` is in no scene's actor list, so it is not changed. `git apply --check` passes, and it builds.

**Revisions:** US NTSC 1.0 only. Others not checked.

**Caveats:**

- Tested by the user on 2026-11-11 (RetroArch, BPS patch), with the
  `shape.rot.x` fix: works. The Ganondorf part was added after that and
  is not tested.
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
- Ganondorf on foot (`En_Viewer` types 7 to 9) is not turned. Zelda's
  horse and Impa in the opening cutscenes are not turned either.

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

## Arrows stay where they appear

**Source diff:** `patches/arrows-stay-where-they-appear.diff`
**Changes:** built on the Mac. Changes the `ovl_En_Arrow` overlay.
**Recorded:** 2026-11-11 00:32:30

**Mechanism:** `EnArrow_Shoot` (`z_en_arrow.c`) starts an arrow with
`Actor_SetProjectileSpeed(&this->actor, 150.0f);` and `this->timer = 12;`.
The patch changes the speed to `0.0f` and the timer to `60`. The arrow
hangs in the air where Link let it go for about 2 seconds (60 frames at
30 fps), then vanishes. This is the OoT version of the ALTTP "Sword beams
stay where they appear" cheat.

**Evidence:** `EnArrow_Fly` turns on gravity (`-0.4`) when
`this->timer < 7.2`, so in its last 7 frames the arrow drops a little.
The arrow's collider is the path it moved in the last frame
(`Player_UpdateWeaponInfo` in `func_809B4800`), and it is cleared when the
arrow doesn't move. `git apply --check` passes, and it builds.

**Revisions:** US NTSC 1.0 only. Others not checked.

**Caveats:**

- Not tested.
- A still arrow hurts nothing. It can only hit an enemy in its last 7
  frames, while it drops.
- The arrow points level (`shape.rot.x` comes from its speed and fall
  speed, both 0 at first).
- Enemy arrows (negative `params`) also stay in place.
- Slingshot seeds and Deku Nuts are not changed.

## Every arrow is an ice arrow

**Source diff:** `patches/every-arrow-is-ice.diff`
**Changes:** built on the Mac. Changes the `ovl_player_actor` overlay.
**Recorded:** 2026-11-11 00:33:50

**Mechanism:** `func_8083442C` (`z_player.c`) spawns the arrow when
Link draws the bow. It gets the arrow type from `func_80834380`
(`ARROW_NORMAL` for the plain bow, `ARROW_NORMAL_HORSE` on a horse,
`ARROW_FIRE`/`ICE`/`LIGHT` for magic arrows). A magic arrow takes magic
with `Magic_RequestChange`, and turns into `ARROW_NORMAL` if Link
doesn't have enough. The patch adds a check right before
`Actor_SpawnAsChild(..., ACTOR_EN_ARROW, ...)`: a normal or horse arrow
becomes `ARROW_ICE`. `EnArrow_Update` then spawns `ovl_Arrow_Ice` on it, as for a
real ice arrow. This is the OoT version of the ALTTP "The sword beam is an ice rod shot" cheat.

**Evidence:** the change is after the magic check, so a plain arrow
becomes an ice arrow with no magic cost. `EnArrow_Init` also gives it the
`DMG_ARROW_ICE` damage flags from its `dmgFlags` table. `git apply
--check` passes, and it builds.

**Revisions:** US NTSC 1.0 only. Others not checked.

**Caveats:**

- Not tested.
- Link still needs the bow and arrows, and each shot uses one arrow. He
  doesn't need the ice Arrows item or any magic.
- A magic arrow shot with too little magic also becomes an ice arrow, in place
  of a plain one.
- Fire, ice and light arrows that Link has the magic for work as before.
- `ovl_Arrow_Ice` calls `Magic_Reset` when it goes away, which sets the
  magic state to `MAGIC_STATE_RESET` and then idle (`Magic_Update`). If
  the Lens of Truth is on, this may turn it off.
- The slingshot and enemy arrows don't change. Horseback archery uses
  the horse arrow type, so its arrows change too.

## Every arrow is a fire arrow

**Source diff:** `patches/every-arrow-is-fire.diff`
**Changes:** built on the Mac. Changes the `ovl_player_actor` overlay.
**Recorded:** 2026-11-11 00:33:52

**Mechanism:** `func_8083442C` (`z_player.c`) spawns the arrow when
Link draws the bow. It gets the arrow type from `func_80834380`
(`ARROW_NORMAL` for the plain bow, `ARROW_NORMAL_HORSE` on a horse,
`ARROW_FIRE`/`ICE`/`LIGHT` for magic arrows). A magic arrow takes magic
with `Magic_RequestChange`, and turns into `ARROW_NORMAL` if Link
doesn't have enough. The patch adds a check right before
`Actor_SpawnAsChild(..., ACTOR_EN_ARROW, ...)`: a normal or horse arrow
becomes `ARROW_FIRE`. `EnArrow_Update` then spawns `ovl_Arrow_Fire` on it, as for a
real fire arrow. This is the OoT version of the ALTTP "The sword beam is a fire rod shot" cheat.

**Evidence:** the change is after the magic check, so a plain arrow
becomes a fire arrow with no magic cost. `EnArrow_Init` also gives it the
`DMG_ARROW_FIRE` damage flags from its `dmgFlags` table. `git apply
--check` passes, and it builds.

**Revisions:** US NTSC 1.0 only. Others not checked.

**Caveats:**

- Not tested.
- Link still needs the bow and arrows, and each shot uses one arrow. He
  doesn't need the fire Arrows item or any magic.
- A magic arrow shot with too little magic also becomes a fire arrow, in place
  of a plain one.
- Fire, ice and light arrows that Link has the magic for work as before.
- `ovl_Arrow_Fire` calls `Magic_Reset` when it goes away, which sets the
  magic state to `MAGIC_STATE_RESET` and then idle (`Magic_Update`). If
  the Lens of Truth is on, this may turn it off.
- The slingshot and enemy arrows don't change. Horseback archery uses
  the horse arrow type, so its arrows change too.

## A Deku Nut casts Din's Fire

**Source diff:** `patches/deku-nut-casts-dins-fire.diff`
**Changes:** built on the Mac. Changes the `ovl_En_Arrow` overlay.
**Recorded:** 2026-11-11 00:34:58

**Mechanism:** a thrown Deku Nut is an `EnArrow` with `params == ARROW_NUT`.
When it hits something, `EnArrow_Fly` (`z_en_arrow.c`) spawns the flash
(`ACTOR_EN_M_FIRE1`), plays `NA_SE_IT_DEKU` and kills the nut. The patch
also spawns `ACTOR_MAGIC_FIRE` (Din's Fire) there, unless the nut is a
cutscene nut (`this->isCsNut`). This is the OoT version of the ALTTP "The
Lamp casts Bombos" cheat.

**Evidence:** Link's own spell uses the same call
(`Player_SpawnMagicSpell`: `Actor_Spawn(..., ACTOR_MAGIC_FIRE, pos, 0, 0,
0, 0)`). `MagicFire_Init` and `MagicFire_Update` take no magic; the magic
cost is paid in `z_player.c` before the spell starts. `git apply --check`
passes, and it builds.

**Revisions:** US NTSC 1.0 only. Others not checked.

**Caveats:**

- Not tested.
- The fire ball is centered on Link, not on the nut. `MagicFire_Update`
  copies `player->actor.world.pos` every frame, as when Link casts it.
- It costs no magic, and Link doesn't need Din's Fire.
- The nut's flash still stuns enemies, as before.
- Din's Fire, Farore's Wind and Nayru's Love share one memory space for
  their code (`ACTOROVL_ALLOC_ABSOLUTE`). Throwing a nut while Nayru's
  Love or Farore's Wind is active may load Din's Fire over it and crash
  the game.
- When the fire goes away, `MagicFire_Destroy` calls `Magic_Reset`. If the
  Lens of Truth is on, this may turn it off.

## Every sword swing makes a spin attack wave

**Source diff:** `patches/every-swing-makes-spin-wave.diff`
**Changes:** built on the Mac. Changes the `ovl_player_actor` and
`ovl_En_M_Thunder` overlays.
**Recorded:** 2026-11-11 00:36:05

**Mechanism:** the glowing ring of a spin attack is the `EnMThunder`
actor. Link spawns it in `func_80837530` (`z_player.c`) with params
`Player_GetMeleeWeaponHeld(this) | magic cost`. When
`PLAYER_STATE2_17` is set at spawn time (a spin that has already started),
`EnMThunder_Init` goes straight to `EnMThunder_SpinAttacking`, the
expanding ring that hits enemies around Link.

The patch has two parts:

- At the end of `func_80837948` (which starts every melee attack), for a
  sword (`Player_GetMeleeWeaponHeld` 1 to 3) and an attack that is not a
  spin (`meleeWeaponAnimation < PLAYER_MWA_SPIN_ATTACK_1H`), it sets
  `PLAYER_STATE2_17`, spawns `ACTOR_EN_M_THUNDER` at Link's waist with
  magic cost 0, and then clears the flag again if it wasn't set before.
- In `EnMThunder_Init`, it removes the `!isMagicAcquired` check, so the
  ring also shows before Link has the magic meter.

This is the OoT version of the ALTTP "The Fighter's Sword shoots beams"
cheat.

**Evidence:** `Actor_Spawn` runs the actor's init function at once, so
the flag only needs to be set around the call. `EnMThunder_Init` clears
`PLAYER_STATE2_17` itself when it starts the ring. With a magic cost of 0,
`Magic_RequestChange` is never called. `git apply --check` passes, and it
builds.

**Revisions:** US NTSC 1.0 only. Others not checked.

**Caveats:**

- Not tested.
- It costs no magic. The ring is the small one (`M_THUNDER_ATTACK_WEAK`),
  not the big charged one.
- No ring shows while the magic state is not idle, for example while the
  Lens of Truth is on or magic is being used.
- Each ring calls `Magic_Reset` when it ends, which may turn off the Lens
  of Truth.
- With the `isMagicAcquired` check gone, the quick spin (roll the stick,
  then B) also shows its ring before Link has the magic meter.
- The Deku Stick and the Hammer don't make rings.
- Swings come fast, so several rings can be on screen at once.

## Nayru's Love is always on

**Source diff:** `patches/nayrus-love-always-on.diff`
**Changes:** built on the Mac. Changes the `ovl_player_actor` and
`ovl_Magic_Dark` overlays.
**Recorded:** 2026-11-11 00:37:18

**Mechanism:** Nayru's Love is the `MagicDark` actor. While it runs,
`MagicDark_DiamondUpdate` (`z_magic_dark.c`) sets
`player->invincibilityTimer = -100` (a negative timer means Link takes no
damage, see `Player_SetInvulnerability`) and counts
`gSaveContext.nayrusLoveTimer` up by 1 each frame. At 1200 it ends. When a
new scene loads, `Player_Init` (`z_player.c`) spawns the spell again if
`nayrusLoveTimer != 0`.

The patch has two parts:

- In `MagicDark_DiamondUpdate`,
  `gSaveContext.nayrusLoveTimer = nayrusLoveTimer + 1;` becomes
  `gSaveContext.nayrusLoveTimer = 1;`, so the timer never reaches 1200
  (or the 1100 point where it starts to flash).
- In `Player_Init`, right before that check, it sets
  `gSaveContext.nayrusLoveTimer = 1` in normal play
  (`gameMode == GAMEMODE_NORMAL` and no cutscene, the same test
  `Play_Init` uses to clear the timer). So the spell starts in every
  scene, from the first one, without Link having it.

This is the OoT version of the ALTTP "The Magic Cape is always on" cheat.

**Evidence:** the diamond's look and sound depend on the timer (below
1100: full color, `NA_SE_PL_MAGIC_SOUL_NORMAL`). The spawn in `Player_Init`
also sets `magicState = MAGIC_STATE_METER_FLASH_1`, as the real spell
does. `git apply --check` passes, and it builds.

**Revisions:** US NTSC 1.0 only. Others not checked.

**Caveats:**

- Not tested.
- Link takes no damage from hits, but some hits may still knock him back,
  as with the real spell. Falling into a pit, drowning, lava, freezing
  water and other hazards still work as before.
- While the spell is on, the magic state is not idle. As with the real
  spell, Link can't cast other spells or shoot magic arrows, and the
  magic meter (if he has one) flashes.
- Songs that play an ocarina effect remove the diamond. It comes back
  after the song (`PLAYER_STATE3_RESTORE_NAYRUS_LOVE`), except after warp
  songs (a known bug in the original, noted in the source), until the next
  scene loads.
- Don't use with "A Deku Nut casts Din's Fire". Both spells share one
  memory space for their code (`ACTOROVL_ALLOC_ABSOLUTE`), so loading Din's
  Fire while Nayru's Love runs may crash.

## Enemies don't knock Link back

**Source diff:** `patches/enemies-dont-knock-link-back.diff`
**Changes:** built on the Mac. Changes the `ovl_player_actor` overlay.
**Recorded:** 2026-11-11 00:39:57

**Mechanism:** `func_808382DC` (`z_player.c`) handles hits on Link. It
calls `func_80837C0C` in two places: for knockback set by an actor
(`this->knockbackType`) and for a hit on Link's body collider
(`this->cylinder.base.acFlags & AC_HIT`). `func_80837C0C` takes the
health, then picks a reaction. For a normal hit on the ground it plays a
stagger animation (and sets `this->speedXZ = 23.0f` for 5 or more damage).
For a large knockback, or when Link is in the air, hanging or climbing, it
throws him with `this->speedXZ = speed` and `this->actor.velocity.y =
yVelocity`. It already has a case with no knockback: when Link is running
(`this->speedXZ > 4.0f`), it only sets `this->unk_890 = 20` (the hurt-run
animation for 20 frames), rumbles, plays the hurt voice and returns.

The patch adds a flag `sNoKnockbackHit`. `func_808382DC` sets it around
its two calls. When the flag is set and Link is not swimming,
`func_80837C0C` now only rumbles and plays the hurt voice, like the
running case but without its hurt-run animation. Link's action, animation,
speed and facing don't change. This is
the OoT version of the ALTTP cheat "Enemies don't knock Link back".

**Evidence:** `func_80837C0C` still takes the health first
(`func_80837B18`) and sets the 20-frame intangibility
(`Player_SetIntangibility`) before the new branch, so damage, death and
the blink after a hit are as before. The frozen (`PLAYER_HIT_RESPONSE_FROZEN`)
and electric (`PLAYER_HIT_RESPONSE_ELECTRIFIED`) branches come before the
new one and still work. Other callers of `func_80837C0C` don't set the
flag: landing from `Player_Action_80844A44`, `Player_StartMode_KnockedOver`,
the electric floor/wall and Ice Trap reactions, and lava/hot floor damage
(the third call in `func_808382DC`, which is outside the flag). `git apply
--check` passes, and it builds.

**Revisions:** US NTSC 1.0 only. Others not checked.

**Caveats:**

- Not tested.
- Link still takes damage and blinks, but he doesn't flinch. Freezing
  (Freezards, ice traps from hits) and electric shocks still stop him,
  since those are not knockback.
- Hits that knock back in a special way, such as an Iron Knuckle swing or
  a boss attack, also no longer throw Link. He keeps standing where he is.
  The fire effect of a fire hit (`func_80838280`) is still applied for the
  actor-knockback path, but the "fall" voice line for those hits may play
  along with the hurt voice.
- Actors that grab or carry Link (Like Like, Wallmaster, Gerudo guards)
  use their own code, not this, and work as before.
- While swimming, hits work as before.

## Enemies always drop a prize

**Source diff:** `patches/enemies-always-drop-a-prize.diff`
**Changes:** built on the Mac. Changes `code` (`z_en_item00.c`).
**Recorded:** 2026-11-11 00:42:21

**Mechanism:** `Item_DropCollectibleRandom` (`src/code/z_en_item00.c`)
picks a random slot (`Rand_ZeroOne() * 16.0f`) in one of the 16-slot drop
tables (`sItemDropIds`). It drops nothing in four cases:

- the slot holds `ITEM00_NONE`;
- the slot holds `ITEM00_FLEXIBLE` and Link needs nothing (enough health,
  magic, ammo, bombs and rupees), so it does `return;`;
- the slot's count in `sDropQuantities` is 0;
- `func_8001F404` turns the item into `ITEM00_NONE` (-1) because Link
  can't use it (bombs with no Bomb Bag, seeds with no Slingshot, arrows with
  no bow, magic with no magic meter).

The patch drops a green rupee in each of these cases (and makes a count of
0 into 1). Everything else, like hearts, magic, fairies when Link is low on
health, and the drops set by `dropFlag`, works as before. This is the OoT
version of the ALTTP cheat "Enemies always drop a prize".

**Evidence:** `ITEM00_NONE` is `0xFF` (`include/z_en_item00.h`), and
`dropId` is a `u8`, so the `-1` from `func_8001F404` compares equal to
it. Counts of 0 are in table 3 (slots 12 to 15) and table 10 (slot 14).
`git apply --check` passes, and it builds.

**Revisions:** US NTSC 1.0 only. Others not checked.

**Caveats:**

- Not tested.
- Everything that uses `Item_DropCollectibleRandom` always drops
  something, not only enemies. That includes grass (`ovl_En_Kusa`), small
  rocks (`ovl_En_Ishi`), bushes and trees (`ovl_En_Wood02`) and hidden
  item spots (`ovl_En_Wonder_Item`). Many enemies call it with no actor,
  so the patch can't tell them apart from grass.
- Enemies that drop items in their own way (not through this function)
  work as before. Some enemies drop nothing at all.
- The extra prizes are mostly green rupees.
- Calls with the `0x8000` flag go to `Item_DropCollectible` and are not
  changed.

## Enemies drop cuccos instead of prizes

**Source diff:** `patches/enemies-drop-cuccos.diff`
**Changes:** built on the Mac. Changes `code` (`z_en_item00.c` and
`z_scene.c`).
**Recorded:** 2026-11-11 00:44:26

**Mechanism:** `Item_DropCollectibleRandom` (`src/code/z_en_item00.c`)
picks an item from the drop tables (`sItemDropIds`). If the result is not
`ITEM00_NONE`, it spawns one or more `ACTOR_EN_ITEM00` items (or a
fairy for `ITEM00_FLEXIBLE` when Link is low on health). The patch spawns
one cucco (`ACTOR_EN_NIW`, params 0, a normal cucco) facing a random way
instead and returns. When the table gives `ITEM00_NONE`, nothing drops,
as before. This is the OoT version of the ALTTP cheat "Enemies drop cuccos
instead of prizes".

A cucco can only spawn when its object `OBJECT_NIW` is loaded
(`Actor_Spawn` checks `Object_GetSlot`), and most scenes don't load it.
So the patch also loads `OBJECT_NIW` in `Object_InitContext`
(`src/code/z_scene.c`), right after `gameplay_keep`, with
`Object_LoadPersistent`. Persistent objects stay loaded for the whole
scene, like `gameplay_keep`, the scene's keep object and Link's object.

**Evidence:** `object_niw` is `0x2540` bytes (9.5 KB, from the map).
The object space is at least 1000 KB. There are 19 object slots
(`ObjectContext.slots[19]`). Normally up to 3 are persistent, and the
largest room object list in the ntsc-1.0 scenes has 15 objects (Lake
Hylia, Market, Death Mountain Trail, Water Temple, Bottom of the Well),
so with the cucco the most used is 19. Only `Scene_CommandObjectList`
adds objects after scene start. In scenes that already list `OBJECT_NIW`
(Kakariko, Lon Lon Ranch), a second copy loads in the room part, and
`Object_GetSlot` finds the persistent copy first. `git apply --check`
passes, and it builds.

**Revisions:** US NTSC 1.0 only. Others not checked.

**Caveats:**

- Not tested.
- Every scene uses 9.5 KB more object memory. If a room was already close
  to the limit, the last objects in it could overflow. This was not
  checked room by room.
- Like the normal drop, many calls give no drop, so not every kill gives a
  cucco.
- Grass, small rocks, bushes and trees (`ovl_En_Kusa`, `ovl_En_Ishi`,
  `ovl_En_Wood02`) and hidden item spots use the same function, so they
  drop cuccos too.
- No fairy, heart or other prize comes from this function any more.
  Enemies that drop items in their own way work as before.
- Hitting a cucco many times calls the attacking cuccos, as usual.
- Not combinable with "Enemies always drop a prize" (both change the same
  lines).

## Enemies face and walk away from Link

**Source diff:** `patches/enemies-face-and-walk-away-from-link.diff`
**Changes:** built on the Mac. Changes `code` (`z_actor.c`).
**Recorded:** 2026-11-11 00:45:45

**Mechanism:** Each frame, `Actor_UpdateAll` (`src/code/z_actor.c`)
sets `actor->yawTowardsPlayer = Actor_WorldYawTowardActor(actor,
&player->actor);` for each actor before it updates. Most enemies turn,
walk and aim with this value. The patch adds `0x8000` (half a turn) to it
for actors in `ACTORCAT_ENEMY`, so they turn their backs to Link and walk
off the other way. This is the OoT version of the ALTTP cheat "Enemies
face and walk away from Link".

Three functions in `z_actor.c` use `yawTowardsPlayer` to check which
way Link is facing, not the enemy: `Player_IsFacingActor`,
`Attention_WeightedDistToPlayerSq` (which actor Z-targeting picks) and
`Attention_ShouldReleaseLockOn`. The patch makes them call
`Actor_WorldYawTowardActor` instead, so Z-targeting and "is Link looking
at me" checks work as before.

**Evidence:** The other uses in `code` check the actor's own facing
(`Actor_IsFacingPlayer`, `Actor_IsFacingAndNearPlayer`, `func_800354B4`
and the head-turn code), and they now act as if Link were behind the
enemy, which fits the cheat. The player uses `yawTowardsPlayer` of the
actor it interacts with or pushes (`z_player.c`), not of enemies. 139
actor overlays read the field. `git apply --check` passes, and it builds.

**Revisions:** US NTSC 1.0 only. Others not checked.

**Caveats:**

- Not tested.
- Enemies that work out the direction to Link in their own way (for
  example `Actor_WorldYawTowardPoint` on Link's position) still come at
  him. Enemies that only check how far Link is still notice him.
- Octoroks and Deku Scrubs turn to `yawTowardsPlayer` before they shoot,
  so they shoot away from Link. Enemies that run away from Link on purpose
  (a hit Deku Scrub, `yawTowardsPlayer + 0x8000`) now run toward him.
- Bosses are in `ACTORCAT_BOSS`, not `ACTORCAT_ENEMY`, and are not
  changed.

## Enemies pass through Link

**Source diff:** `patches/enemies-pass-through-link.diff`
**Changes:** built on the Mac. Changes `code` (`z_collision_check.c`).
**Recorded:** 2026-11-11 00:46:48

**Mechanism:** OC ("object collision") is the collider check that pushes
actors apart so they don't overlap. `CollisionCheck_OC`
(`src/code/z_collision_check.c`) tests each pair of OC colliders, unless
`CollisionCheck_Incompatible` returns true (the OC types don't match, or
both belong to the same actor). The patch makes
`CollisionCheck_Incompatible` also return true when one collider belongs
to Link (`ACTORCAT_PLAYER`) and the other to an enemy
(`ACTORCAT_ENEMY`). Then enemies and Link walk through each other. This
is the OoT version of the ALTTP cheat "Enemies pass through Link".

**Evidence:** Damage doesn't use OC. Enemy attacks hit Link with AT/AC
checks (`CollisionCheck_AT`), and Link's sword hits enemies the same
way, so both still work. Enemies still collide with each other and with
other actors. `git apply --check` passes, and it builds.

**Revisions:** US NTSC 1.0 only. Others not checked.

**Caveats:**

- Not tested.
- Some enemies use an OC touch with Link to grab him. These no longer
  grab: Like Like (`ovl_En_Rr`) and the small Floormasters
  (`ovl_En_Floormas`). Armos (`ovl_En_Am`) use an OC touch to wake up
  and to stop when they bump into Link, so they may act differently.
- Bosses (`ACTORCAT_BOSS`) still push Link.

## Enemies spawn three at a time and come back after you clear a room

**Source diff:** `patches/enemies-spawn-copies-and-come-back.diff`
**Changes:** built on the Mac. Changes `code` (`z_actor.c`).
**Recorded:** 2026-11-11 00:48:18

**Mechanism:** When a room loads, `Actor_SpawnEntry`
(`src/code/z_actor.c`) spawns each actor in the room's actor list with
`Actor_Spawn`. The patch checks the new actor, and if it is in
`ACTORCAT_ENEMY`, it spawns two more of the same actor with the same
position, rotation and params. So each enemy in a room's list comes as
three.

`Actor_Spawn` also refuses to spawn any enemy in a room whose clear flag
is set: `((profile->category == ACTORCAT_ENEMY) &&
Flags_GetClear(play, play->roomCtx.curRoom.num))`. The patch removes this
part, so enemies come back in those rooms too. This is the OoT version of
the ALTTP cheat "Overworld enemies spawn several copies and come back after
you kill them".

**Evidence:** In OoT most enemies already come back when Link leaves a
room and comes back. Only rooms with the clear flag set
(`Flags_SetClear`) keep their enemies away. That flag is set after some
fights, for example Big Octo (`ovl_En_Bigokuta`), the Deku Scrub puzzle
(`ovl_En_Hintnuts`), the ice shutter rooms (`ovl_Bg_Ice_Shutter`) and
boss rooms. The flag itself is still set, so doors and other things that
read it stay open. Gold Skulltulas (`ACTOR_EN_SW`) are not copied, since
each copy would carry the same token flag. `git apply --check` passes,
and it builds.

**Revisions:** US NTSC 1.0 only. Others not checked.

**Caveats:**

- Not tested.
- The three copies start at the same spot. Most push each other apart
  (OC), but enemies that don't move or have no OC collider may stay on
  top of each other and look like one.
- Mini-bosses and scripted enemies come as three too (Stalfos, Lizalfos,
  Iron Knuckles, Poe Sisters, Dead Hand and so on). Their cutscenes,
  music and room-clear checks were made for one, so some fights may act
  strangely or not end. Dark Link is in `ACTORCAT_BOSS` and is not
  copied.
- Enemies that the game spawns later (Stalchildren in Hyrule Field,
  enemies that come out of the ground) are not copied. Only room actor
  lists are.
- Big Octo and other enemies held back by the clear flag come back. Some
  of them may also check their own flags.
- Three times the enemies use more memory and CPU. Rooms with many enemies
  may slow down, and if memory runs out some enemies don't spawn.
- Bosses (`ACTORCAT_BOSS`) are not copied.

## Castle guards move at double speed

**Source diff:** `patches/castle-guards-move-at-double-speed.diff`
**Changes:** built on the Mac. Changes the `ovl_En_Heishi1` and
`ovl_En_Heishi3` overlays.
**Recorded:** 2026-11-11 00:49:58

**Mechanism:** `ovl_En_Heishi1` is the patrolling guard in the Hyrule
Castle courtyard (`hairal_niwa`, day and night). Each guard copies one
row of `sAnimParamsInit` into `animSpeed`, `moveSpeedTarget`,
`moveSpeedMax`, `bodyTurnSpeedTarget`, `bodyTurnSpeedMax` and the
head-turn values. `EnHeishi1_Walk` walks along the path with
`Math_ApproachF(..., this->moveSpeed)`. When a guard sees Link,
`EnHeishi1_MoveToLink` walks to him with `Math_ApproachF(&this->moveSpeed,
6.0f, 1.0f, 0.4f)`, turning at up to `3000.0f`.

The patch doubles the walk animation speed, the move speed and its step,
and the body turn speed and its step, in all four `sAnimParamsInit` rows
(`1.0f, ..., 3.0f, 0.5f, 1000.0f, 200.0f` to `2.0f, ..., 6.0f, 1.0f,
2000.0f, 400.0f`, and `3.0f, ..., 6.0f, 0.8f, 2000.0f, 400.0f` to
`6.0f, ..., 12.0f, 1.6f, 4000.0f, 800.0f`). It does the same in
`EnHeishi1_SetupMoveToLink` (animation `3.0f` to `6.0f`) and
`EnHeishi1_MoveToLink` (`6.0f, 0.4f` to `12.0f, 0.8f`, and `3000.0f,
300.0f` to `6000.0f, 600.0f`). The head-turn values are not changed, so
the guards look around at the usual speed.

`ovl_En_Heishi3` is the standing guard in the castle grounds
(`spot15`). When it sees Link, `EnHeishi3_CatchStart` makes it walk
toward him for 20 frames with `this->actor.speed = 2.5f` and walk
animation speed `1.0f`. The patch makes these `5.0f` and `2.0f`. This is the OoT version of the
ALTTP cheats "Soldiers chase Link at double speed" and "Soldiers move
faster".

**Evidence:** The field order is from `z_en_heishi1.h` (the second value,
`-10.0f` or `-3.0f`, is the animation morph and is not changed).
`Math_ApproachF` doesn't overshoot, so the guards still reach each
waypoint; the waypoint check is within 20 units. `ACTOR_EN_HEISHI1` is
only in the `hairal_niwa` and `hairal_niwa_n` actor lists, and
`ACTOR_EN_HEISHI3` only in `spot15`. `git apply
--check` passes, and it builds.

**Revisions:** US NTSC 1.0 only. Others not checked.

**Caveats:**

- Not tested.
- OoT guards don't fight. When one sees Link, the game shows a message and
  throws him out, so the faster guards mainly make the courtyard
  sneaking harder.
- Other guards (`ovl_En_Heishi2` and `ovl_En_Heishi4`, in the market,
  Kakariko and other places) are NPCs that stand still or talk, and are
  not changed.

## Octoroks spit bombs

**Source diff:** `patches/octoroks-spit-bombs.diff`
**Changes:** built on the Mac. Changes the `ovl_En_Okuta` overlay.
**Recorded:** 2026-11-11 00:52:25

**Mechanism:** `EnOkuta_SpawnProjectile` (`ovl_En_Okuta/z_en_okuta.c`)
spawns the rock as another `ACTOR_EN_OKUTA` with params `0x10`, 25 units
in front of the Octorok, and it flies straight at Link. The patch spawns a
lit bomb (`ACTOR_EN_BOM`, `BOMB_BODY`) instead, 50 units in front and
20 units up, and lobs it at Link: `velocity.y = 12.0f` and
`speed = (xzDistToPlayer - 50.0f) / 22.0f`, so it comes down about 22
frames later near where Link stands. The dust puff and the spit sound are
the same as before. This is the OoT version of the ALTTP cheat "Octoroks
spit something other than rocks".

**Evidence:** `ovl_En_Bom` uses `OBJECT_GAMEPLAY_KEEP`, so it can spawn
anywhere. `EnBom_Move` moves the bomb with `Actor_MoveXZGravity`
(gravity `-1.2f`, set every frame in `EnBom_Update`), bounces it off
walls and slows it down on the ground. Rotation x and z are 0, since
`EnBom_Init` uses `shape.rot.z` for extra explosion damage. A bomb
explodes at once when its OC collider touches an enemy, so it spawns 50
units out, past the Octorok's collider (radius 20). Octoroks shoot when
Link is 160 to 560 units away (`EnOkuta_WaitToShoot`), so the speed is
about 5 to 23. The spit comes on frame 6 of `EnOkuta_Shoot`, while the
Octorok is jumping, so the bomb starts above the water. `git apply --check` passes, and it builds.

**Revisions:** US NTSC 1.0 only. Others not checked.

**Caveats:**

- Not tested. The arc was worked out from the code, not seen, so the bombs
  may land a bit short or long, and on higher or lower ground.
- The bomb has the usual 70-frame fuse. Link can pick it up and throw it
  back, which kills the Octorok.
- A bomb that falls into deep water (`depthInWater >= 20.0f`) goes out,
  as usual, so bombs that fall short into the water don't explode.
- The bomb explosion hurts Link, enemies and the Octorok itself, and
  breaks walls and rocks like a normal bomb.
- Big Octo (`ovl_En_Bigokuta`) is a different actor and is not changed.

## Every enemy moves with the speed of the enemy before it in the list

**Source diff:** `patches/enemies-move-with-previous-enemys-speed.diff`
**Changes:** built on the Mac. Changes `code` (`z_actor.c`).
**Recorded:** 2026-11-11 00:53:28

**Mechanism:** Most actors move with `Actor_MoveXZGravity` or
`Actor_MoveXYZ` (`src/code/z_actor.c`). These turn `actor->speed` and
the world rotation into a velocity in `Actor_UpdateVelocityXZGravity`
(`actor->velocity.x = actor->speed * Math_SinS(actor->world.rot.y);` and
so on) and `Actor_UpdateVelocityXYZ`. The patch adds a small function
`Actor_GetMoveSpeed`. For an actor in `ACTORCAT_ENEMY` with an
`actor->prev`, it returns `actor->prev->speed`; for all other actors it
returns `actor->speed`. The two velocity functions use it in place of
`actor->speed`. So each enemy moves in its own direction, but at the
speed of the enemy before it in the enemy list. This is the OoT version of
the ALTTP cheat "Every enemy moves with the speed of the sprite in the
previous slot".

**Evidence:** Each actor category has its own linked list
(`actorCtx->actorLists`, `Actor.prev` and `Actor.next` in
`include/actor.h`), so `prev` of an enemy is always another enemy.
`Actor_AddToCategory` puts each new actor at the head of the list, so
`prev` is the enemy that spawned just after it. The newest enemy has no
`prev` and keeps its own speed. The enemy's own `speed` value is not
changed, only how far it moves. `git apply --check` passes, and it
builds.

**Revisions:** US NTSC 1.0 only. Others not checked.

**Caveats:**

- Not tested.
- An enemy after one that stands still can't walk (it can still jump or
  fall). An enemy after a fast one runs fast. A charging or jumping enemy
  (Tektite, Wolfos lunge) may barely move if the enemy before it is slow.
- Enemies that move by their animation (`Actor_UpdatePosByAnimation`) or
  set their position directly are not changed.
- Any other actor that is in `ACTORCAT_ENEMY` (for example parts or
  shots that an enemy spawns in that category) is in the same list, so it
  gets a neighbor's speed too.

## Every enemy pushes the enemy after it instead of moving itself

**Source diff:** `patches/enemies-push-the-next-enemy.diff`
**Changes:** built on the Mac. Changes `code` (`z_actor.c`).
**Recorded:** 2026-11-11 00:54:23

**Mechanism:** `Actor_UpdatePos` (`src/code/z_actor.c`) moves an actor by
its velocity: `actor->world.pos.x += (actor->velocity.x * speedRate) +
actor->colChkInfo.displacement.x;`, and the same for y and z.
`Actor_MoveXZGravity` and `Actor_MoveXYZ` call it, and so do 52 places
in the actor overlays. The patch adds the x and z velocity of an enemy
(`ACTORCAT_ENEMY`) to the position of `actor->next`, the next enemy in
the enemy list, instead of to its own. The enemy still moves itself up and
down (gravity, jumps) and still gets its own collision push
(`colChkInfo.displacement`). The last enemy in the list has no `next`
and moves itself as usual. This is the OoT version of the ALTTP cheat
"Every enemy pushes the sprite in the next slot instead of moving itself".

**Evidence:** `next` is the next actor of the same category
(`include/actor.h`). `Actor_AddToCategory` puts each new actor at the
head of the list, so `next` is the enemy that spawned just before it, and
the first enemy spawned is the last in the list. `git apply --check`
passes, and it builds.

**Revisions:** US NTSC 1.0 only. Others not checked.

**Caveats:**

- Not tested.
- `Actor_UpdateAll` saves `prevPos` at the start of each actor's update,
  after the enemy before it has already pushed it. So the wall check of the
  pushed enemy doesn't see the push, and enemies may be pushed through
  walls or off ledges.
- An enemy far away in the room can push one next to Link, so enemies may
  slide around with no walking animation.
- Enemies that move by animation or set their position directly don't
  push.
- Any other actor in `ACTORCAT_ENEMY` (parts or shots an enemy spawns in
  that category) is in the same list and pushes or gets pushed too.
- The diff applies together with "Every enemy moves with the speed of the
  enemy before it" (checked with `git apply --check`). That pair was not
  built.

## Items are collected anywhere in the room

**Source diff:** `patches/items-are-collected-anywhere.diff`
**Changes:** built on the Mac. Changes `code` (`z_en_item00.c`).
**Recorded:** 2026-11-11 00:55:53

**Mechanism:** `EnItem00_Update` (`src/code/z_en_item00.c`) gives an
item (rupees, hearts, bombs, arrows, sticks, nuts, seeds, magic, small
keys, Heart Pieces and so on) only when Link is close:
`(this->actor.xzDistToPlayer <= 30.0f) && (this->actor.yDistToPlayer >=
-50.0f) && (this->actor.yDistToPlayer <= 50.0f)`. Items that use a
get-item id (`getItemId`) are then offered with
`Actor_OfferGetItemNearby` (`Actor_OfferGetItem` with a range of 50 by
10). The patch makes the first check `100000.0f` in every direction, and
offers get-item items with `Actor_OfferGetItem(..., 100000.0f,
100000.0f)`. So every loaded item is collected as soon as it can be, from
anywhere. This is the OoT version of the ALTTP cheat "Items are collected
anywhere on the screen".

**Evidence:** Without the second change, a far item with a get-item id
would pass the first check but never be offered, and it would stay where
it is (`EnItem00_Update` returns until `Actor_HasParent`). The
collected-item timer `unk_154` (35 frames) still keeps an item from being
collected twice. `git apply --check` passes, and it builds.

**Revisions:** US NTSC 1.0 only. Others not checked.

**Caveats:**

- Not tested.
- Only `ovl_En_Item00` items (drops and placed collectibles) are
  changed. Gold Skulltula tokens, chests and items given by NPCs work as
  before.
- Items placed on high ledges or behind walls in the loaded rooms are
  collected right away, including Heart Pieces and small keys. That can
  skip puzzles.
- Get-item items still need Link to be free (`Actor_OfferGetItem` checks
  that he isn't climbing, hanging, holding a bomb and so on), and Link
  plays the get-item pose for Heart Pieces and keys wherever he is.
- When several items are out, the get-item ones are taken one at a time.

## Lift every rock without the Goron's Bracelet or gauntlets

**Source diff:** `patches/lift-every-rock-without-strength.diff`
**Changes:** built on the Mac. Changes the `ovl_player_actor` overlay.
**Recorded:** 2026-11-11 00:57:22

**Mechanism:** Link's strength upgrade is checked in three places in
`ovl_player_actor/z_player.c`:

- `func_8083A0F4` (the lift): for a bomb flower (`ACTOR_EN_BOMBF`) or an
  `ACTOR_EN_KUSA` with `Player_GetStrength() <=
  PLAYER_STR_NONE`, Link plays the "can't pull" animation
  (`gPlayerAnim_link_normal_nocarry_free`, `Player_Action_80846408`)
  instead of lifting.
- The A button lift: a silver rock (`ACTOR_EN_ISHI` with type 1) returns
  0 when `strength < PLAYER_STR_SILVER_G`.
- The A button against a wall: the large block `ACTOR_BG_HEAVY_BLOCK`
  returns 0 when `Player_GetStrength() < PLAYER_STR_GOLD_G`.

The patch removes all three checks, so Link lifts small rocks, silver
rocks, the golden gauntlet blocks and bomb flowers with no strength
upgrade. This is the OoT version of the ALTTP cheat "Lift every rock
without the Power Glove".

**Evidence:** `Player_GetStrength` (`src/code/z_player_lib.c`) is only
used in these three places, in the child Goron's Bracelet model, and in
`ovl_Obj_Oshihiki` (push blocks). `ovl_En_Ishi` and `ovl_En_Bombf` offer
the carry (`Actor_OfferCarry`) without checking strength, and the bomb
flower spawns its bomb when Link takes it (`EnBombf_GrowBomb`).
`git apply --check` passes, and it builds.

**Revisions:** US NTSC 1.0 only. Others not checked.

**Caveats:**

- Not tested.
- The pause screen still shows no bracelet or gauntlets, and child Link
  doesn't wear the bracelet.
- Push blocks that need strength (`ovl_Obj_Oshihiki`) still need it.
- Child Link can now lift silver rocks and the golden gauntlet blocks.
  The normal game never lets him, so those lifts may look wrong for child
  Link.
- Bronze boulders (`ovl_Obj_Hamishi`) are broken with the hammer, not
  lifted, and are not changed.

## Bushes and rocks stay in place after Link lifts them

**Source diff:** `cheats/patches/bushes-and-rocks-stay-after-lifting.diff`
**Changes:** built on the Mac. Changes the `ovl_En_Ishi` and
`ovl_En_Kusa` overlays.
**Recorded:** 2026-11-11 00:59:51

**Mechanism:** When Link picks up a rock, `EnIshi_Wait`
(`src/overlays/actors/ovl_En_Ishi/z_en_ishi.c`) sees
`Actor_HasParent(&this->actor, play)` and calls
`EnIshi_SetupLiftedUp(this)`. The rock actor itself goes into Link's
hands, so its spot is empty. Bushes do the same in `EnKusa_Main`
(`ovl_En_Kusa/z_en_kusa.c`) with `EnKusa_SetupCarried(this)`.

The patch adds an `Actor_Spawn` of the same actor (`ACTOR_EN_ISHI` or
`ACTOR_EN_KUSA`) at that point, with the lifted actor's `home`
position, `home` rotation and `params`. So a new rock or bush appears
where the old one stood, and Link carries the old one as usual. This is
the OoT version of the ALTTP cheat "Bushes and rocks stay in place after
Link lifts them".

Type 1 bushes (`ENKUSA_TYPE_1`) get no copy. In `EnKusa_Fall` they
already go to `EnKusa_SetupUprootedWaitRegrow` after a throw and grow
back at home, so a copy would leave two bushes there.

**Evidence:** `EnIshi_Wait` and the `Actor_HasParent` branch of
`EnKusa_Main` are the only places where each actor starts its carried
state. `EnKusa_Fall` kills types 0 and 2 and regrows type 1. The rock
and bush objects are already loaded when the original is in the room, so
`Actor_Spawn` finds them. `git apply --check` passes, and it builds.

**Revisions:** US NTSC 1.0 only. Others not checked.

**Caveats:**

- Not tested.
- Each copy is a full rock or bush, so it can be lifted again. Rocks and
  bushes that drop items or spawn bugs (En_Ishi params bit 4) do it every
  time, so this gives as many drops and bugs as you want.
- A large (silver) rock sets its switch flag when it breaks, and
  `EnIshi_Init` removes a large rock whose flag is set. The copy stays
  until Link leaves the room; after that, the rock is gone as before.
- Type 1 bushes still grow back after a short wait instead of staying at
  once.

## Pots stay in place after Link lifts them

**Source diff:** `cheats/patches/pots-stay-after-lifting.diff`
**Changes:** built on the Mac. Changes the `ovl_Obj_Tsubo` overlay.
**Recorded:** 2026-11-11 01:00:41

**Mechanism:** When Link picks up a pot, `ObjTsubo_Idle`
(`src/overlays/actors/ovl_Obj_Tsubo/z_obj_tsubo.c`) sees
`Actor_HasParent(&this->actor, play)` and calls
`ObjTsubo_SetupLiftedUp(this)`. The pot actor goes into Link's hands, so
its spot is empty.

The patch adds an `Actor_Spawn` of `ACTOR_OBJ_TSUBO` just before that
call, with the lifted pot's `home` position, `home` rotation and
`params`. A new pot appears where the old one stood, and Link carries
the old one as usual. This is the OoT version of the ALTTP cheat "Pots
stay in place after Link lifts them", and works like "Bushes and rocks
stay in place after Link lifts them".

**Evidence:** `ObjTsubo_Idle` is the only place the pot starts its
carried state. `ObjTsubo_Init` snaps the new pot to the floor and finds
its object (`sObjectIds`, chosen by params bits 8) the same way as the
first pot, and the object is already loaded. No other actor spawns
`ACTOR_OBJ_TSUBO`. `git apply --check` passes, and it builds.

**Revisions:** US NTSC 1.0 only. Others not checked.

**Caveats:**

- Not tested.
- Only plain pots (`ovl_Obj_Tsubo`). Flying pots (`ovl_En_Tubo_Trap`)
  and other liftable things are not changed.
- Each copy can be lifted again. A pot whose item has no collectible flag
  drops it each time it breaks. A pot with a collectible flag (params bits
  9-14) drops its item only once, since `EnItem00_Init` removes an item
  whose flag is already set.

## No music

**Source diff:** `cheats/patches/no-music.diff`
**Changes:** built on the Mac. Changes `code` (the audio driver part,
`src/audio/internal/effects.c`).
**Recorded:** 2026-11-11 01:02:11

**Mechanism:** `Audio_SequenceChannelProcessSound`
(`src/audio/internal/effects.c`) works out each sequence channel's
volume:

```c
channelVolume = channel->volume * channel->volumeScale * channel->seqPlayer->appliedFadeVolume;
...
channel->appliedVolume = channelVolume * channelVolume;
```

Every note's velocity is `layer->velocitySquare2 * channel->appliedVolume`,
set only in this function. The patch sets `channelVolume` to 0 for every
sequence player except `SEQ_PLAYER_SFX`. So the background music
(`SEQ_PLAYER_BGM_MAIN`, `SEQ_PLAYER_BGM_SUB`) and fanfares
(`SEQ_PLAYER_FANFARE`) still run but make no sound, and sound effects
play as usual. This is the OoT version of the ALTTP cheat "No music".

**Evidence:** `SEQ_PLAYER_*` is in `include/sequence.h`
(`BGM_MAIN` 0, `FANFARE` 1, `SFX` 2, `BGM_SUB` 3). `playerIdx` is set
in `AudioLoad_SyncInitSeqPlayerInternal`. `appliedVolume` is written
only here and read only for note velocity. The music keeps running, so
game code that waits for a fanfare to end (item get, song learned)
works as before. `git apply --check` passes, and it builds.

**Revisions:** US NTSC 1.0 only. Others not checked.

**Caveats:**

- Not tested.
- Link's ocarina, and songs that NPCs play on the ocarina, use the sound
  effect player (`SFX_CHANNEL_OCARINA`), so they are still heard.
- Area sounds that play as a sequence on the BGM player, such as the
  nature ambience in Hyrule Field at night, are silent too.

## Every song plays with the next instrument

**Source diff:** `cheats/patches/songs-play-with-next-instrument.diff`
**Changes:** built on the Mac. Changes `code` (the audio driver part,
`src/audio/internal/seqplayer.c`).
**Recorded:** 2026-11-11 01:03:09

**Mechanism:** When a sequence channel runs its "set instrument" command,
`AudioSeq_SetInstrument` (`src/audio/internal/seqplayer.c`) handles
synthetic waves (`instId >= 0x80`), drums (`0x7F`) and sound effects
(`0x7E`) on their own. Any other id is a normal instrument in the
channel's sound font:

```c
if ((channel->instOrWave = AudioSeq_GetInstrument(channel, instId, &channel->instrument, &channel->adsr)) == 0) {
```

The patch adds 1 to `instId` in that last case, for every sequence
player except `SEQ_PLAYER_SFX`. If the font has no instrument at the new
id (`Audio_GetInstrumentInner` returns NULL), it uses instrument 0
instead, so the channel is not left silent. So each music part plays
with the next instrument in its sound font, and drums and synthetic
waves stay as they are. This is the OoT version of the ALTTP cheat
"Every song plays with the next instrument".

**Evidence:** `AudioSeq_SetInstrument` is called only from the channel
command handler in `seqplayer.c`. `Audio_GetInstrumentInner`
(`src/audio/internal/playback.c`) returns NULL when the id is
`>= numInstruments` or the slot is empty; it also sets
`gAudioCtx.audioErrorFlags`, which only the debug display reads. `git
apply --check` passes, and it builds.

**Revisions:** US NTSC 1.0 only. Others not checked.

**Caveats:**

- Not tested.
- Sound effects, Link's ocarina and NPC ocarina playback use the sound
  effect player, so they keep their instruments.
- The next instrument may be a very different sound (a drum kit sample,
  a voice) or play in a very different range, so some parts may sound
  odd or nearly silent.
- When the last instrument in a font is set, the part uses instrument 0
  of that font instead.

## Music and sound effects play 2 whole tones higher

**Source diff:** `cheats/patches/sound-plays-4-semitones-higher.diff`
**Changes:** built on the Mac. Changes `code` (the audio driver part,
`src/audio/internal/playback.c`).
**Recorded:** 2026-11-11 01:04:03

**Mechanism:** Each audio update, `Audio_ProcessNotes`
(`src/audio/internal/playback.c`) works out the playback rate of every
playing note:

```c
sampleStateAttrs.frequency *= playbackState->vibratoFreqScale * playbackState->portamentoFreqScale;
sampleStateAttrs.frequency *= gAudioCtx.audioBufferParameters.resampleRate;
```

The patch multiplies it by 1.2599211 (2 to the power 4/12) on the next
line. Every note plays 4 semitones (2 whole tones) higher, and a little
faster for one-shot samples. This covers music, sound effects, voices,
drums and the ocarina, since they all go through this code. This is the
OoT version of the ALTTP cheat "Music and sound effects play 2 whole
tones higher".

**Evidence:** Both paths before this line (a note being released, which
uses `attrs->freqScale`, and a playing note, which uses
`layer->noteFreqScale`) end at these two lines, so the change covers
both. `Audio_NoteSetResamplingRate` turns the result into the sample
step. `git apply --check` passes, and it builds.

**Revisions:** US NTSC 1.0 only. Others not checked.

**Caveats:**

- Not tested.
- `Audio_NoteSetResamplingRate` limits the rate to about 4 times the
  sample rate. A note that already played above about 3.2 times its
  sample rate is held at 4 times, so it is less than 4 semitones higher.
- Samples play faster, so sound effects and voices are a little shorter.
  Note lengths in the music don't change, so the tempo stays the same.

## Every sound effect comes from the wrong sound list

**Source diff:** `cheats/patches/sfx-come-from-wrong-list.diff`
**Changes:** built on the Mac. Changes `code` (the audio part,
`src/audio/game/sfx.c` and `general.c`).
**Recorded:** 2026-11-11 01:05:42

**Mechanism:** A sound effect id (`include/sfx.h`) is the bank in bits
12-15, flags in bits 9-11 and the index in the bank in bits 0-8
(`SFX_BANK`, `SFX_INDEX`). The bank picks which list of sounds in
sequence 0 plays it: player, item, environment, enemy, system, ocarina
or voice. `Audio_PlaySfxGeneral` (`src/audio/game/sfx.c`) queues the id
as it is.

The patch adds `Audio_WrongSfxListId`, which moves an id to the next
bank (player to item, item to environment, environment to enemy, enemy
to system, system to voice, voice to player) and keeps the flags. The
index is taken modulo the new bank's size (`NA_SE_*_END -
(NA_SE_*_BASE + 1)`), so it always names a real sound. The ocarina
bank stays as it is. `Audio_PlaySfxGeneral` calls it on every id, so
each sound effect plays the sound at the same place in another list.
This is the OoT version of the ALTTP cheat "Every sound effect comes
from the wrong sound list".

`Audio_StopSfxByPosAndId`, `Audio_StopSfxByTokenAndId`,
`Audio_StopSfxById` and `Audio_IsSfxPlaying` call it too, so game
code that stops a looping sound, or waits for one to end, finds it under
its new id. `func_800F4C58` (`src/audio/game/general.c`, used for the
timer sound in `src/code/z_actor.c`) looks for a playing sound by id in
`gSfxBanks`, so it uses the new id for that search too.

**Evidence:** `SFX_BANK`, `SFX_INDEX`, the `BANK_*` enum and the
`NA_SE_*_BASE`/`_END` values are in `include/sfx.h`. Every sound
effect call (`Sfx_PlaySfxCentered`, `Actor_PlaySfx`, `func_800F4*`
and so on) ends in `Audio_PlaySfxGeneral`. Outside `sfx.c`, only
`func_800F4C58` compares an id against `gSfxBanks` (the other uses in
`general.c` take a bank and entry index). `git apply --check` passes,
and it builds.

**Revisions:** US NTSC 1.0 only. Others not checked.

**Caveats:**

- Not tested.
- Ocarina sounds (`BANK_OCARINA`, such as Link's ocarina notes) are
  left as they are, since the ocarina's notes are set through its own
  channel and would be silent in another bank. Nothing is moved into the
  ocarina bank.
- Some sounds in the new list are long loops (for example environment
  sounds). A short sound that becomes a loop plays until the game stops
  the old id, or until the bank entry is pushed out by other sounds.
- Bank sizes differ, so many ids in a large bank (enemy has the most)
  land on the same few sounds of a small bank.
- System sounds (menu cursor, text) become voice bank sounds, so the
  pause menu and text boxes make voice sounds.

## Text appears almost at once

**Source diff:** `cheats/patches/text-is-near-instant.diff`
**Changes:** built on the Mac. Changes `code` (`src/code/z_message.c`).
**Recorded:** 2026-11-11 13:37:54

**Mechanism:** `Message_DrawText` (`src/code/z_message.c`) draws the
decoded text up to `msgCtx->textDrawPos` each frame, then moves on by one
character:

```c
if (msgCtx->textDelayTimer == 0) {
    msgCtx->textDrawPos = i + 1;
    msgCtx->textDelayTimer = msgCtx->textDelay;
} else {
    msgCtx->textDelayTimer--;
}
```

The patch replaces this with the scan that the game's own "quick text"
code (`MESSAGE_QUICKTEXT_ENABLE`) uses. It moves `textDrawPos` straight
to the next stop code: `MESSAGE_AWAIT_BUTTON_PRESS`, `MESSAGE_BOX_BREAK`,
`MESSAGE_BOX_BREAK_DELAYED`, `MESSAGE_END`, `MESSAGE_TEXTID`,
`MESSAGE_EVENT`, `MESSAGE_PERSISTENT`, `MESSAGE_OCARINA`,
`MESSAGE_FADE` or `MESSAGE_FADE2`. So each part of a text box appears in
one frame. The stop code itself is not passed over: on the next frame,
`textDrawPos` moves one more step onto it, so it runs as usual. The same
change is made in `Message_DrawTextWide` (Japanese text).

**Evidence:** `MESSAGE_AWAIT_BUTTON_PRESS` and `MESSAGE_OCARINA` only
run when they are the newest character (`i + 1 == msgCtx->textDrawPos`),
which is why the scan stops before them. The other stop codes return from
the draw loop when it reaches them. Codes such as `MESSAGE_SFX`,
`MESSAGE_ITEM_ICON` and the choice codes run when they are drawn, with no
such check. The scan skips the argument byte of `MESSAGE_SHIFT` like the
game's own scan, and it stops at `decodedTextLen`. The control codes are
in `include/message_data_fmt.h`. `git apply --check` passes, and it
builds.

**Revisions:** US NTSC 1.0 only. Others not checked.

**Caveats:**

- Not tested.
- Text speed codes (`MESSAGE_TEXT_SPEED`) no longer slow the text down.
  Waits that are part of the message (`MESSAGE_BOX_BREAK_DELAYED`,
  `MESSAGE_FADE`) still wait as before.
- An argument byte of another control code (for example a `MESSAGE_SFX`
  id) can have the same value as a stop code. Then the text stops there
  for one frame. The game's own quick text has the same behavior.
- All text that goes through `Message_DrawText` is changed, so the
  end credits and other text shown by the message system appear at once
  too. Text is still shown one box at a time, and you still press A to
  go on.
