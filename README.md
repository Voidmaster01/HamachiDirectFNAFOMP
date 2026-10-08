# HamachiDirect

# This was made using AI, Claude.ai but we made sure to check all code within it. If you want to yell at us make an issue post

A [MelonLoader](https://melonwiki.xyz) mod that lets you play a Unity Netcode game privately over a **Hamachi** (VPN) network, skipping the game's online matchmaking servers.

One player hosts, everyone else joins by typing in the host's Hamachi address. No cloud lobby, no relay.

> Parts of this project (such as the UI elements) were made with AI. Everything was reviewed by a systems engineer for safety.

**Version:** 2.0.0  **Author:** voidcrew

---

## Features

- **Direct host / join** over a VPN adapter using Unity Transport (UTP)
- **Binds only to the VPN adapter** when hosting, so the game isn't exposed on your other network interfaces
- **Auto-copies your Hamachi address** to the clipboard when you host
- **In-game popup** (matched to the game's menu style) for entering the host address, with paste support and masked digits so the IP doesn't leak on stream
- **First-run guide** that explains the workflow and must be acknowledged before the mod's actions unlock
- **Locks the three online buttons** (Join Lobby, Host Public Lobby, Host Private Lobby) so nobody accidentally opens the game to strangers
- **Player cap** that removes players beyond the configured limit
- **Hides the "failed to connect to Unity Services" overlay** on the matchmaking screen
- **Diagnostics** written to the MelonLoader log: adapter list, ping, UDP probe, and filtered networking lines from Unity's `Player.log`. IP addresses are masked.

## Requirements

- [MelonLoader](https://melonwiki.xyz) installed in the game
- [LogMeIn Hamachi](https://vpn.net) running, with everyone joined to the **same network**

## Installation

1. Install MelonLoader into the game (run the game once so it generates its folders).
2. Install Hamachi and create a tunnel to your friends (free up to 5)
2. Grab `HamachiDirect.dll`.
3. Drop the DLL into the game's `Mods/` folder.
4. Launch the game. The first-run guide appears on the main menu.

## How to use

Everyone must have Hamachi **on** and be in the same Hamachi network.

### Host (one player)

1. On the main menu, press **PLAY ONLINE**
2. Press **F9**. You'll enter the lobby and your Hamachi address is copied to the clipboard.
3. Send that address to your friends in a **private** message.

### Join (everyone else)

1. Press **PLAY ONLINE**.
2. Press **F8**, paste the host's address with **Ctrl+V**, then press **Enter** to save.
3. Press **F10** to join. You'll land in the host's lobby.

### Hotkeys

| Key | Action |
|-----|--------|
| **F7** | Show the guide again |
| **F8** | Open/close the settings popup (host IP) |
| **F9** | Host |
| **F10** | Join |
| **F11** | Stop / leave |

In the popup: **Delete** clears the field, **Enter** saves, **Ctrl+V** pastes, **F8** cancels.

> **Do not press** the game's own *Join Lobby*, *Host Public Lobby*, or *Host Private Lobby* buttons. They use the game's online servers rather than your private Hamachi link, and could open your game to strangers. The mod greys them out by default.

## Configuration

Settings live in `UserData/MelonPreferences.cfg` under the `HamachiDirect` category.

| Key | Default | Description |
|-----|---------|-------------|
| `Port` | `7777` | Game UDP port. A small probe listener uses `Port + 1`. |
| `VpnPrefix` | `25.` | Address prefix used to find the Hamachi adapter and validate the host IP. |
| `HostIp` | *(empty)* | Host's Hamachi address (set via F8). Joiners only. |
| `MaxPlayers` | `5` | Extra players beyond this are disconnected. |
| `SkipGameApproval` | `true` | Turns off Netcode connection approval for direct play, since the game's approval handler isn't set up in this mode. Restored when you stop. |
| `LockOnlineButtons` | `true` | Greys out the three cloud-matchmaking buttons. |

## How it works

- **Hosting:** finds the adapter whose IPv4 address starts with `VpnPrefix`, sets the transport to listen on it, calls `NetworkManager.StartHost()`, then loads the Lobby scene and presses the lobby's Update button, since the game's own host button normally does this.
- **Joining:** validates the saved IP (IPv4, must start with `VpnPrefix`), points the transport at it, raises connect attempts to 30 x 1000 ms, and calls `StartClient()`. A watcher reports success, refusal, or timeout.
- **Cleanup:** F11 uses the game's own `LeaveGame()` when in a session, then restores the original approval and transport settings.
- **UDP probe:** the host runs a tiny UDP echo on `Port + 1`. Joiners send it a short ping so the log can tell a blocked tunnel apart from a game-level failure.

## Troubleshooting

- **"No VPN adapter found"**: start Hamachi and make sure it's connected.
- **"Press PLAY ONLINE first"**: the mod needs the game's network manager, which exists once you're on the online screen.
- **Can't connect**: check the MelonLoader console. Look for the `Ping to host` and `UDP probe` lines. If both fail, it's a Hamachi/firewall problem rather than the game.
- **Address copy failed**: copy it manually from your Hamachi client.
- **Popup unavailable**: edit `UserData/MelonPreferences.cfg` directly.

## Security notes

- Share the host address privately. Never post it on stream or in public chats.
- With `SkipGameApproval` on, anyone who can reach the host's Hamachi address and port can connect, so only add people you trust to your Hamachi network. The player cap limits headcount but is not authentication.
- Logs mask IP addresses, but check them before sharing.

## Project layout

Single file: `Core.cs` (`HamachiDirect.Core : MelonMod`). It contains input handling, the UI (guide, popup, toast), networking, the probe listener and the log tail.

## Disclaimer

This is an unofficial community mod and is not affiliated with the game's developers. Use at your own risk, and online play may be restricted by the game's terms of service.
