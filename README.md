# NinjaCombatComponent

**A data-driven, multiplayer-ready melee combat plugin for Unreal Engine 5**

Developed by **Darex Dynamic Systems**

---

## Overview

NinjaCombatComponent is a complete melee combat framework for Unreal Engine 5, built around a data-driven DataAsset workflow. No C++ required to configure — every system is tuned directly in the editor. Designed for competitive action games in the style of Naruto: Ultimate Ninja Storm, Boruto: Shinobi Striker, and arena fighters.

The plugin covers the full combat loop out of the box:

- **Combo chains** with buffered input and animated notify windows
- **Single actions** for launcher and special attacks
- **Block & Guard Meter** with chip damage and guard break
- **Parry system** with configurable reaction window
- **Dash with I-frames** and cancel rules
- **Air combat** — launch, juggle routing, and drift control
- **Knockdown & get-up** with quick rise option
- **Hit stop** (freeze frame on impact)
- **Directional hit reactions** (Front / Back / Left / Right)
- **Weapon trail FX** via Niagara
- **Target Assist** for close-range aim correction
- **Lock-On** with camera control and target switching
- **AI combat brain** for enemy characters
- **Attribute system** — Health, Chakra, Stamina, Strength, Level/XP
- **Full multiplayer replication** — server-authoritative by default

---

## Engine & Requirements

| | |
|---|---|
| **Engine** | Unreal Engine 5.7 |
| **Language** | C++ (plugin source included) |
| **Dependencies** | Niagara (bundled with UE5) |
| **Network** | Multiplayer-ready, server-authoritative |

---

## Components

| Component | Purpose |
|---|---|
| `NinjaCombatComponent` | Attacker-side: combos, actions, block, dodge, hit feel, target assist |
| `NinjaDamageReceiverComponent` | Victim-side: hit stun, launch, knockdown, reactions, ragdoll |
| `NinjaAttributeComponent` | Health / Chakra / Stamina / Strength / Level / XP |
| `NinjaMovementComponent` | Sprint, walk, multi-jump, slide, movement dash |
| `NinjaLockOnComponent` | Target lock, camera control, target switching |
| `NinjaAnimInstance` | AnimBP base class — reads replicated state from all components |

---

## DataAssets

All gameplay tuning lives in DataAssets — no code changes needed:

| DataAsset | Configures |
|---|---|
| `NinjaCombatComboDataAsset` | Combo chain steps, montages, damage, FX per step |
| `NinjaCombatActionDataAsset` | Single attack (launcher, special), montage, damage, FX |
| `NinjaBlockDataAsset` | Block montage, Guard Meter, parry window, chip damage |
| `NinjaDodgeDataAsset` | Dash distance, montage, I-frame window, cooldown, cancel rules |
| `NinjaHitReactionDataAsset` | Reaction montages per direction and HitStrength |
| `NinjaAirCombatDataAsset` | Juggle steps, drift velocity, air combo routes |
| `NinjaMovementDataAsset` | Jump count, sprint speed, slide settings |

---

## Montage Setup (AnimNotifyStates)

All timing windows are placed directly inside montages as colored bars — no DataAsset entry required:

| Color | NotifyState | Function |
|---|---|---|
| 🟢 Green | `Ninja Combat - Combo Window` | Window in which buffered input continues the combo |
| 🔴 Red | `Ninja Combat - Hit Trace` | Active hit detection window. Configure socket, radius in Details |
| 🔵 Blue | `Ninja Combat - Swing FX` | Weapon trail visibility window. Niagara system overridable per-notify |

---

## Quick Setup

### 1 — Add Components
Add the following to your Character Blueprint:
- `NinjaCombatComponent`
- `NinjaDamageReceiverComponent`
- `NinjaAttributeComponent`
- *(optional)* `NinjaMovementComponent`, `NinjaLockOnComponent`

### 2 — Create a Combo DataAsset
- Create a `NinjaCombatComboDataAsset`
- Set a `ComboTag` (e.g. `Combat.Combo.LightAttack`)
- Add the asset to `NinjaCombatComponent → Combos`

### 3 — Set Up Montages
For each combo step montage, add:
- **Hit Trace** NotifyState with valid socket names
- **Combo Window** NotifyState for input buffering

### 4 — Wire Inputs
```
Attack Input  →  Started  →  PressActionByTag(ComboTag)
Block Input   →  Started  →  PressBlock()
Block Input   →  Completed →  ReleaseBlock()
Dodge Input   →  Started  →  PressDodge()
Lock-On Input →  Started  →  ToggleLockOn()
Right Stick   →  Triggered →  SwitchTarget(StickInput)
```

### 5 — Connect the Damage Pipeline
In your Character Blueprint:
```
NinjaDamageReceiver → OnHitProcessed → NinjaAttribute → ApplyDamage
NinjaAttribute → OnDeath → [your respawn / game-over logic]
```

---

## Multiplayer

The plugin is multiplayer-first. Critical systems are server-authoritative by default.

**Handled automatically:**
- Combat montages broadcast via Multicast RPC from server
- Hit detection runs on server only (`HasAuthority()` guard)
- Health / Chakra / Stamina replicated via `UPROPERTY(Replicated)`
- Death & ragdoll via `Multicast_OnDeath`
- Lock-On state replicated with `OnRep`
- Guard Meter replicated ReadOnly
- `VictimState` (HitStun / Launched / KnockedDown) replicated

**What you need to configure:**
- `bServerAuthoritative = true` on `NinjaCombatComponent` *(default)*
- `bReplicates = true` on your Character
- `CharacterMovementComponent → SetIsReplicated(true)` in constructor

> **Dedicated Server:** Niagara, sound, and camera shake FX are gated behind `IsNetMode(NM_DedicatedServer)` checks and do not run on the server.

---

## Debug Tools

All debug visualization is off by default. Enable in the Details panel during PIE — no code changes required.

| Component | Flag | Shows |
|---|---|---|
| `NinjaCombatComponent` | `bDebugTargetAssist` | Detection cone and acquire sphere |
| `NinjaLockOnComponent` | `bDebugDrawLockOn` | Acquire/break radii, LOS trace, target info |
| `NinjaLockOnComponent` | `bDebugDrawSwitchRadius` | Switch candidate search sphere |
| `NinjaAttributeComponent` | `bDebugDrawStats` | Floating HP/Chakra/Stamina bars (local client only) |
| `NinjaDamageReceiverComponent` | `bDebugPrintToScreen` | On-screen hit receive events |
| `NinjaDamageReceiverComponent` | `bDebugLaunch` | Launch impulse arrow + velocity verification |
| `NinjaDamageReceiverComponent` | `bDebugDirectionalReactions` | Direction selection with dot product values |
| `NinjaMovementDataAsset` | `bDebugMode` | Jump/sprint/slide state change events |

---

## Known Limitations

- No built-in respawn system — death is permanent until you wire your own respawn logic
- AI characters on non-Pawn collision channels are not found by lock-on acquisition
- `HitBoneName` parameter on `ApplyDamage` does not yet have a Blueprint-facing tooltip

---

## License

This plugin is proprietary software by **Darex Dynamic Systems**.  
All rights reserved.
