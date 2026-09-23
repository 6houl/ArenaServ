# Endless Online 0.3.X  [ sfx usage ]

# files inside the root of /sfx folder are no longer used by the client engine and are now custom-only.


# compatibility note 

# client 0.3.x will use  /sfx/client sfx/ctrl sfx/effect and sfx/instr for all sound playback
# sounds in the /sfx root are still (and only) used by Quest PlaySound rule, and looped Map hotspots.

# as a result most sounds in the /sfx root are no longer used;
# it is possible to 'recycle' those unused sounds in the root of /sfx/ with caution!
# leaving all sounds in the root of /sfx/ as-is will work fine and requires no effort at all.


# new folders

# /sfx/client - used for main interface sounds
# /sfx/ctrl - used for most melee attacks
# /sfx/effect - used for (magic) effects
# /sfx/inst - used and pitched for instruments


