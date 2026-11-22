# FFXI Addon — CurePlease_Addon

> [!NOTE]
> **Quick Reference**
> - **Path:** `C:\Windower\addons\CurePlease_Addon\`
> - **Author / Version:** Daniel_H / v1.2 Windower
> - **Commands:** `//cpaddon`
> - **Load Status:** On-Demand (unloaded)
> - **Origin:** Third-Party (`Daniel_H` — CurePlease companion addon)

UDP telemetry bridge that streams party buff bitfields and player spellcasting completion/interruption events to the external **CurePlease** Windows healer application over `127.0.0.1:19769`.

## What the Code Does
- Uses LuaSocket (`socket.udp()`) with a 1s timeout to send plain-text UDP datagrams to `ip:port` (defaults: `127.0.0.1:19769`).
- Hooks incoming `0x076` (Party Buffs) in `Run_Buff_Function()`: iterates the 5 party-member slots (`k = 0..4`, 48-byte stride), unpacks each member's ID (`data:unpack('I', k * 48 + 5)`), resolves character name via `windower.ffxi.get_mob_by_id()`, decodes up to 32 active 10-bit buff IDs (8 low bits + 2 high bits from the packed mask bytes), and transmits `"CUREPLEASE_buffs_<Name>_<id1>,<id2>,..."` over UDP.
- Hooks `action` events for the local player (`data.actor_id == player.id`):
  - `category == 4`: sends `"CUREPLEASE_casting_finished"`.
  - `category == 8`: sends `"CUREPLEASE_casting_interrupted"` (`param == 28787`) or `"CUREPLEASE_casting_blocked"` (`param == 24931`).

## Commands & Usage
- `//cpaddon settings <ip> <port>` — Updates target UDP IP and port at runtime.
- `//cpaddon check` — Prints the currently configured IP and port to chat.
- `//cpaddon verify` — Sends `"CUREPLEASE_confirmed"` UDP packet to test connectivity.
- `//cpaddon cmd <command>` — Sends `"CUREPLEASE_command_<command>"` to CurePlease.

## Events, Packets & Technical Hooks
- **Windower Events:** `incoming chunk`, `addon command`, `action` (`category == 4` and `8`)
- **Packets:**
  - Incoming `0x076` (Party Buffs): decoded for 5 party members and up to 32 buff IDs each.
- **Dependencies / Libraries:** `socket` (LuaSocket UDP), `packets`

## Local Settings & Customizations
- No `data/settings.xml`; `127.0.0.1:19769` is hardcoded at the top of `CurePlease_Addon.lua` (overrideable per session via `//cpaddon settings`).

## Related Notes
- `FFXI Windower Overview`
- `FFXI Windower Addons Catalog`
