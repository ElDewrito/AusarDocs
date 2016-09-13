# Engine Globals

Many of the following globals have retained similar functionality throughout each iteration of the Halo engine. They can be obtained from TLS or static locations within the executable's `.DATA` section.

## Version 1.114.4592.2

#### TLS

The `Offset` in the table below is relative to the TLS slot 0 base address in the game's main thread. The `Allocation Source` contains the image address where the allocation occurs; this can be used as a starting point for further investigation if needed.

Name | Offset | Allocation Source | Type | Comments
--- | --- | --- | --- | ---
observer globals | `0x8` | `0x1406B850E` | `struct` |
orientations | `0x110` | `0x140684045` | `pool` | |
scripted camera globals | `0x190` | `0x1406B850E` | `struct` |
director globals | `0x198` | `0x1406BCB47` | `struct` |
composer lights globals | `0x1B8` | `0x1406DCA4E` | `struct` |
composer globals | `0x1C8` | `0x1407015AE` | `struct` |
usability globals | `0x1198` | `0x14073966E` | `struct` |
actively dissolving objects | `0x11F0` | `0x140761492` | `struct` |
player effects | `0x1218` | `0x14076DADE` | `struct` |
simulated_input | `0x1228` | `0x14077348E` | `struct` |
fp weapons | `0x1260` | `0x140779A1E` | `struct` |
accumulators state | `0x1268` | `0x14077FA30` | `struct` |
campaign meta-game globals | `0x1270` | `0x14078D4B2` | `struct` |
campaign meta-game globals secondary | `0x1278` | `0x14078D551` | `struct` |
game_state_header | `0x1280` | | `struct` | game globals starts at offset `0x8`
game allegiance globals | `0x1288` | `0x14087905E` | `struct` | |
game heat globals | `0x1290` | `0x140883DEE` | `struct` | |
game save globals | `0x12A0` | `0x1408865EB` | `struct` | |
game time globals | `0x12A8` | `0x14088715B` | `struct` | |
DOF data | `0x1310` | `0x1408C4121` | `struct` | |
PlayerFocus | `0x1320` | `0x1408CE0D1` | `struct` | |
player control globals | `0x1340` | `0x1409DE0C7` | `struct` | |
player control globals deterministic | `0x1348` | `0x1409DE167` | `struct` | |
player mapping globals | `0x1350` | `0x1409E57CE` | `struct` | |
rumble | `0x1358` | `0x1409EA2C2` | `struct` | |
player training globals | `0x1360` | `0x1409EB21E` | `struct` | |
players globals | `0x1370` | `0x140A04362` | `struct` | |
players per map globals | `0x1378` | `0x140A042D5` | `struct` | |
crew orders | `0x1398` | `0x140A069F2` | `struct` | |
crew orders nav | `0x13A0` | `0x140A06A91` | `struct` | |
game engine globals | `0x13A8` | `0x140A0E79E` | `struct` | |
local game engine globals | `0x13B0` | `0x140A0E854` | `struct` | |
game engine render globals | `0x13B8` | `0x140A0E8E1` | `struct` | |
multiplayer_sound globals | `0x13C0` | `0x140A1DFA2` | `struct` | |
spawn influencer globals | `0x2C20` | `0x140A260D0` | `struct` | |
incident globals | `0x2C28` | `0x140A4AD59` | `struct` | |
hs designer strings | `0x2C30` | `0x140A5EAF1` | `array` | |
random math | `offset` | `0x140B317FB` | `struct` | |
micro system state | `0x2C40` | `0x140B52782` | `struct` | |
micro systems (update region) | `0x2C50` | `0x140B52826` | `struct` | |
micro systems (shared region) | `0x2C58` | `0x140B528B6` | `struct` | |
micro systems (MP update region) | `0x2C48` | `0x140B52947` | `struct` | |
collision hierarchy hash table | `0x2C80` | `0x140E96083` | `struct` | |
collision hierarchy globals | `0x2C78` | `0x140E96110` | `struct` | |
havok gamestate | `0x2C88` | `0x140EA6BC7` | `struct` | |
havok contact point globals | `0x2C90` | `0x140EB725E` | `struct` | |
impact globals | `0x2D28` | `0x140EC58BE` | `struct` | |
physics constants | `0x2D30` | `0x140EC63A0` | `struct` | |
Ragdolls State | `0x2D38` | `0x140EC79AD` | `struct` | |
scenario interpolator states | `0x2D50` | `0x140EE59DE` | `struct` | |
kill trigger volume state | `0x2D58` | `0x140EE62AE` | `struct` | |
sky objects | `0x2D60` | `0x140EE86CD` | `struct` | |
soft surface globals | `0x2D68` | `0x140EE938B` | `struct` | |
game sound globals | `0x2D78` | `0x140EF99F4` | `struct` | |
game sound globals main | `0x2D88` | `0x140EF9A80` | `struct` | |
game sound scripted impulses | `0x2D80` | `0x140EF9B0D` | `struct` | |
game music storage | `0x2D90` | `0x140EF9B9A` | `struct` | |
deterministic game sound globals | `0x2DA0` | `0x140F01D2E` | `struct` | |
game sound player effects globals | `0x2DA8` | `0x140F0294B` | `struct` | |
sound classes | `0x2DC0` | `0x140F0554E` | `struct` | |
scripted sound placement storage | `0x2DD0` | `0x140F0BD62` | `struct` | |
sound state restore manager | `0x2DE0` | `0x140F158CA` | `struct` | |
forced visible objects | `0x2E08` | `0x140F28550` | `struct` | |
forced visible lights | `0x2E00` | `0x140F285EF` | `struct` | |
visibility active portals | `0x2E10` | `0x140F2B66E` | `struct` | |
ai globals | `0x2E40` | `0x141975B59` | `struct` | |
valid dialog events state | `0x2E60` | `0x1419791C2` | `struct` | |
vocalization timers | `0x2E68` | `0x14197D1B2` | `struct` | |
Musketeer Avatars | `0x2E78` | `0x14198B8F5` | `struct` | |
task records | `0x2F10` | `0x1419F0D3A` | `struct` | |
ai player state globals | `0x2F18` | `0x1419F5EBE` | `struct` | |
swarm_spawner | `0x2F38` | `0x141A0947A ` | `struct` | |
spawner_globals | `0x2F40` | `0x141A09506` | `struct` | |
vocalization megaqueue | `0x2F48` | `0x141A0B982` | `struct` | |
rasterizer game states | `0x2FE0` | `0x141AE5CAE` | `struct` | |
hue saturation control | `0x2FF8` | `0x141AE5DCC` | `struct` | |
userGraphicsScalingOptions | `0x3050` | `0x141B1BDC5` | `struct` | |
render texture globals | `0x3058` | `0x141B1E6CB` | `struct` | |
scripted exposure globals | `0x3060` | `0x141B2D34B` | `struct` | |
interaction ripples | `0x4960` | `0x141B5CDCE` | `struct` | |
Forge atmosphere override settings | `0x4998` | `0x141B6FC9B` | `struct` | |
rasterizer | `0x49A0` | `0x141B7C8C0` | `struct` | |
render game globals | `0x49A8` | `0x141B7DEBE` | `struct` | |
DOF globals | `0x49B0` | `0x141B7E39C` | `struct` | |
fp orientations | `0x4A10` | `0x1421F6067` | `struct` | |
campaign objectives | `0x4A30` | `0x14221618E` | `struct` | |
object name list | `0x4B20` | `0x142BF031E` | `struct` | |
OBJ: Render Data | `0x4B38` | `0x142BF7E79` | `struct` | |
object messaging queue | `0x4B28` | `0x142BF7F44` | `struct` | |
object placement globals | `0x4B58` | `0x142C5DD4C` | `struct` | |
object scripting | `0x4B60` | `0x142C6572E` | `struct` | |
object early movers | `0x4B70` | `0x142C74A43` | `struct` | |
Power up effect service | `0x4B88` | `0x142C96E72` | `struct` | |
object schedule globals | `0x4B98` | `0x142C977AE` | `struct` | |
malleable property service | `0x4BA0` | `0x142C9AB32` | `struct` | |
big battle squads | `0x4BA8` | `0x142C9B972` | `struct` | |
big battle squad units | `0x4BB0` | `0x142C9BA12` | `struct` | |
damage manager | `0x4BB8` | `0x142CA1702` | `struct` | |
spartan tracking service | `0x4BC8` | `0x142CA2D18` | `struct` | |
recycling_volumes | `0x4BD8` | `0x142CAFDAA` | `struct` | |
cloth globals | `0x4C00` | `0x142D00B6A` | `struct` | |
object globals | `0x4C08` | `0x142D418EB` | `struct` | |
object render globals | `0x4C10` | `0x142D4198E` | `struct` | |
objects | `0x4B18` | `0x142BF7E19` | `pool` | |

#### Static

Name | Address | Type | Description
--- | --- | --- | --- 
camera_fov | `0x1458ECF90` | `float` | Camera field of view
kill_volumes_disable | `0x145B1CB90` | `bool` | Disable kill volumes
revive_enabled | `0x144735FD1` | `bool` | Enables revive on a global level
revive_enabled_mp | `0x1459FBAB3` | `bool` | Enables multiplayer revive
havok components | `0x145AD60C8` | `array` | 
havok contact points | `0x145AD60C0` | `array` | 
sound tracker data | `0x145B3B4C0` | `array` | 
sound instance data | `0x145B3B550` | `array` | 
delayed sound instance data | `0x145B3B4E0` | `array` | 
campaigns | `0x145F691C0` | `array` | 
campaign missions | `0x145F691C8` | `array` | 
campaign levels | `0x145F691D0` | `array` | 
campaign insertions | `0x145F691D8` | `array` | 
multiplayer levels | `0x145F691C0` | `array` | 
textures | `0x14643BE90` | `array` | 
dynamic render targets | `0x1464ADAB0` | `array` | 
simulation view data array | `0x146EA79E8` | `array` | 
tag instances | `0x1462F97A0` | `array` | 
