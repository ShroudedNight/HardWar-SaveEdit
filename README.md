# HardWar Savegame Editor
This is a savegame editor for the cyberpunk retro game HardWar. This editor was created as an experiment on the highly dynamic data structures of its savegame file. While it mostly serves as a savegame parser,
it does have the ability to edit basic information like pilot cash, moth health/shields, and hangar cash.

## Instructions
To use this editor, simply navigate to the deployment page [Here](https://julianozelrose.github.io/HardWar-SaveEdit/). You can begin editing a savegame by clicking 'Open', then selecting your savegame file.
Note that this editor only supports UIM 6 (Steam release or latest patch) savegames. Use the tab control to navigate between moths, pilots and hangars. Use the dropdown to select the moth, hangar, or pilot you wish to edit. Click 'Save' when you are done,
and the editor will generate a new savegame file based on the modifications made.

![HardWar-SaveEdit-UI](https://github.com/user-attachments/assets/4231fb54-d1d0-440b-82b3-8f08ad45f905)


## Savegame Data Structures
The data structures of the HardWar savegame are highly dynamic. The pilot entity list start offset is pointed to by the
pointer stored on `0xDF8`. The hangar entity list start offset is pointed to by the pointer stored on `0xDE0`. The moth
entity list is pointed to by the pointer stored on `0xDD4`. Pilot entities are `0x37C` in size, hangar entities are
`0x964` in size, and moth entities are `0x448` in size. Offsets relevant to the savegame entity lists can be found in the
table below.

| **Offset**         | **Type**     | **Description**             |
| :---               | :---         | :---                        |
| 0xDD0              | DWORD        | Moth entity count           |
| 0xDD4              | DWORD        | Moth entity list pointer    |
| 0xDD8              | DWORD        | Moth pointer list pointer   |
| 0xDDC              | DWORD        | Hangar entity count         |
| 0xDE0              | DWORD        | Hangar entity list pointer  |
| 0xDE4              | DWORD        | Hangar pointer list pointer |
| 0xDF4              | DWORD        | Pilot entity count          |
| 0xDF8              | DWORD        | Pilot entity list pointer   |
| 0xDFC              | DWORD        | Pilot pointer list pointer  |

### Moth Entity List
The moth entity list start is pointed to by the pointer stored on `0xDD4`. With each moth data structure being `0x37C` in size, this
value can be used as an iterator to step through the moth entity list. Stored at the end of the moth data structure is
a pointer to the dynamic address of the next moth in the entity list. When a moth is destroyed, it is removed from the
moth entity list. So the moth "array" is more than likely a vector, rather than an array.

### Hangar Entity List
The hangar entity list is stored after the moth entity list. The location of the hangar entity list start is stored on
offset `0xDE0`. With the offset of the first hangar in the entity list, it is then possible to step through the hangar
entity list using the iterator `0x964`.

### Pilot Entity List
After the hangar entity list is the final entity list; pilots. The total pilot entity count is stored on offset `0xDF4`.
The start of the pilot entity list is pointed to by the vaue stored on offset `0xDF8`. The value stored on offset `0xDFC`
points to the list of dynamic pilot addresses. When a pilot "dies", they are teleported to the Limbo! hangar.

## Offset Tables
### Moth
| **Offset** | **Type** | **Description**      |
| :---       | :---     | :---                 |
| 0x1D0      | UInt32   | Hangar               |
| 0x1DC      | UInt32   | Moth Type            |
| 0x294      | Int32    | Shields              |
| 0x298      | Int32    | Engine Damage        |
| 0x29C      | Int32    | Structure Damage     |
| 0x2A0      | Int32    | CPU Damage           |
| 0x2A4      | Int32    | Power Damage         |
| 0x2A8      | Int32    | Weapons Damage       |
| 0x2DC      | UInt32   | Pilot                |
| 0x2E0      | UInt32   | Passenger            |

For the moth data structure itself, the value on offset `0x1D0` points to the hangar that the moth is currently parked in. If the moth is not in a hangar, then it becomes a null pointer.
Similarly, the values on offsets `0x2DC` and `0x2E0` are both pointers that point to the pilot and the passenger, respectively. If there is no pilot or passenger, they also become null pointers.
The value stored on offset `0x1DC` stores the "type" of moth. The remaining offsets hold the values of shields and damage.

### Hangar
| **Offset** | **Type**  | **Description** |
| :---       | :---      | :---            |
| 0x010      | String    | Display Name    |
| 0x048      | UInt32    | Owner           |
| 0x8BC      | Int32     | Cash Held       |
| 0x8D8      | UInt32    | Bay 1           |
| 0x8DC      | UInt32    | Bay 2           |
| 0x8E0      | UInt32    | Bay 3           |
| 0x8E4      | UInt32    | Bay 4           |
| 0x8E8      | UInt32    | Bay 5           |
| 0x8EC      | UInt32    | Bay 6           |

The hangar data structure's values on offset `0x010` points to the owner of the hangar. If the hangar is owned by a pilot, it will point to the pilot's dynamic address. If the hangar is owned by
a faction, it will point to the location of the faction's headquarters (i.e. "Police HQ"). The values stored on offsets `0x8D8` - `0x8EC` point to the moth that is currently residing in the respective
hangar bay. If the hangar bay is empty, the pointer is null.

### Pilot
| **Offset** | **Type**  | **Description** |
| :---       | :---      | :---            |
| 0x004      | String    | Name            |
| 0x02C      | UInt32    | Status          |
| 0x030      | UInt32    | Location        |
| 0x03C      | Int32     | Cash            |
| 0x040      | UInt32    | Pilot Type      |
| 0x298      | UInt32    | Faction         |

For the pilot data structure, the value on offset `0x030` represents the location of the pilot. If the pilot is in a moth, it will point to the moth's dynamic address. If the pilot is on foot in a hangar,
then it will point to the dynamic address of the hangar. The value on offset `0x298` represents the faction that the pilot belongs to. If a pilot belongs to a faction, it will point to the dynamic address
of the faction's headquarters. If the pilot is not a member of a faction, it will be a null pointer.

## Credits
- Special thanks to [ShroudedNight](https://github.com/ShroudedNight) for improving the editor's pilot and moth parsing ability.
