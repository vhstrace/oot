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
