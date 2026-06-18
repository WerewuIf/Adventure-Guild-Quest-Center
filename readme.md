# Adventure Guild Quest Center — Setup & User Guide

## What it does

Automates your Adventurer's Guild quests end to end: accepting quests, fighting the monsters, looting the kills, donating items, and finishing quests — all while you do work on other pages. It also does your Battle Pass hunt quest.

This guide covers setup, the main toggles, and the less obvious behavior (like Battle Pass filling) that isn't visible just from clicking around the UI.

---

## Requirements

- You must be in **[AHAB] Casual Ahab** with the **[AHAB]** tag in your display name. The script checks this on load and will block automation if you aren't verified.
- You can change your name by going to any manga page's comment section.
- Install **[Tampermonkey](https://www.tampermonkey.net/)** for your browser.

---

## Installation

1. Install Tampermonkey.
2. Open Tampermonkey → click the **+** to create a new script.
3. Paste the entire script and save.
4. The script is now active on the site. You don't need to do anything else for it to load.

You can also use the import function in **Settings**, once the script is running, instead of repeating this process per browser/profile.

---

## Setup (do this once)

1. **Go to the Adventurer's Guild page.**
2. **Select your quests.** Each quest row now has a small checkbox on the left. Tick every quest you want automated.
3. **Click "Add selected"** in the toolbar that appears under the page header. This is the step that actually saves the quest into autocomplete — ticking the checkbox alone only marks it as *queued*, it doesn't run yet.
4. **Turn on Autokills** (toggle in the same toolbar, or in Settings → General). This is the master switch — without it, the script will still accept/donate/finish quests for you, but it will never attack anything.
5. **Turn on Prekills** (optional, recommended). Lets the script work on several kill quests at once instead of strictly one at a time. See the dedicated section below for what this actually changes.
6. **(Recommended) Click the 🔒 button** next to the status pill at the bottom of the screen. This only appears on the tab currently doing the real work. See "The lock system" below for what it actually does — it's not the same thing as the automation lock itself.
7. Unpause script if necessary.
   
That's it. The script runs on its own from here. Everything past this point is reference material for understanding *what* it's doing and why.

---
**Get started:**


https://github.com/user-attachments/assets/92d4366f-804a-4ec1-bba8-c48af4538c04

* Reload the tabs to transfer the automation lock if needed.
* The lock icon only appears on the specific tab holding the automation lock.
* Clicking the lock icon triggers a warning beofre page reloads to prevent accidental navigation and lock loss (not visible in the video for some reason).
* The video also contains locations of the debug export and reset buttons.
  

A explanation for some features can be found below.

---

## Prekills: running multiple quests at once

By default (Prekills off), the script finishes one kill quest completely — including waiting for every kill to be confirmed dead and looted — before starting the next which is **very slow**. 

With **Prekills on**, the script instead:

- Attacks targets for *every* eligible quest, not just the active one, as soon as a wave has usable monsters for them.
- Defers looting on quests you haven't accepted yet — kills are tracked and held, then claimed the moment quest gets accepted.
- Lets quests progress in parallel instead of sitting idle while one quest waits for a wave to respawn or a kill to confirm dead.

This is just pure optimization. It requires **Autokills** to be on, and turning Autokills off will also turn Prekills off automatically.

---

## Battle Pass auto-fill ("BP fill")

This is the one feature that isn't visible anywhere in the UI flow, so it's worth explaining explicitly.

Guild quests spend a lot of time *waiting* — for a new monster wave to spawn, for a kill to be confirmed dead, for loot to register. Rather than sit idle during those gaps, the script checks whether your Battle Pass **hunt** quest still needs kills, and if so, spends the dead time attacking Battle Pass hunt targets instead.

How it decides what to do:

- If a guild quest has work available, the script always does that first — Battle Pass filling only happens when guild-side work is genuinely unavailable right now (e.g checking for monsters).
- A guild quest that's blocked waiting on you to donate more items, or to get a class for active skill quest counts as "no work available" for this purpose — the script keeps filling Battle Pass time instead of repeatedly retrying a donate it can't complete or being stuck.
- It only fills the **hunt** progress. The Battle Pass **stamina** quest can't be advanced directly — it will complete as you spend more stamina.
- Default hunt targets are **Lizardman Flamecaster** and **Lizardman Shadowclaw**. You can add, remove, or reset these in **Settings → Waves → Battle Pass hunt targets**.
- The moment a guild quest has actionable work again, the script hands control back to it — Battle Pass filling is always interruptible, never blocking.
- If you'd rather the script never touch the Battle Pass at all, enable **Skip Battle Pass** in **Settings → General**. Guild quests will keep running normally; Battle Pass toasts and automation simply stop. Toggling this on mid-hunt aborts cleanly back to guild quests (or idle) rather than leaving anything stuck mid-attack.

You'll see this reflected as a "Battle Pass hunt" toast with progress (e.g. `3/8`) appearing and disappearing between your guild quest toasts — that's expected, it means the script is filling gaps rather than wasting time.

If all adventure quests are completed, bp runs normally.

---
## Loot Finder: Kills That The Script Didn't Make Itself

Normally the script only credits a kill toward a quest if it itself attacked that monster. **Loot Finder** (Settings → General) closes a specific gap: monsters you've already damaged past a quest's minimum requirement through some other means — manual attacks, slayers bot — that are sitting and un-looted in a wave's graveyard.

With it on, the script periodically scans graveyards across all wave sources for any dead monster matching an enabled kill quest's target name where your damage already clears that quest's minimum. When it finds one, it's credited and looted exactly as if the script had attacked and killed it directly.

- If the quest is already active/accepted, the kill counts immediately and gets queued for looting right away.
- If the quest is only available but not yet accepted, the kill is held as deferred — same as a Prekills deferred kill — and gets claimed automatically the moment you accept the quest.
- It runs as its own, independent of whether Autokills is on.
- Supports stacked monsters created by slayer's bot.
-  Scan frequency is **Loot finder scan interval** in Settings → Network → Polling intervals. 

The loot finder can also help recover loot from a hard reset

---
## Tab Lock: Multi-Tab Safety & Coordination

Only one tab acts as the **"worker"** for your account at a time—attacking, looting, accepting, donating, and finishing quests all happen from a single tab. This mechanism is known as the **tab lock**, and it exists to prevent two open tabs from ever trying to attack the same monster or accept the same quest at once.

#### How It's Acquired

* **Automated Request:** Requested automatically on page load — no manual action is required to acquire it.
* **Account-Scoped Isolation:** The lock is also scoped per account (per player ID), meaning two different accounts open in different tabs will never compete for the same lock.
* **Silent Queueing:** If the lock is free, the tab grabs it instantly. If another tab already holds it, the new tab queues silently in the background and is automatically granted the lock the moment the holder releases it (e.g., when the holding tab is closed, reloaded, or navigated away).

#### What the Lock Controls

* **Core Actions:** Attacking, looting, and quest actions (`accept` / `donate` / `finish`) only ever execute on the lock-holding tab.
* **🔒 "Main Tab" Button:** This button is only visible on the tab that currently holds the lock; it is hidden everywhere else. Clicking it does not affect the lock status directly—instead, it arms a browser-native *"Leave this page?"* confirmation dialog on that specific tab. This prevents you from accidentally reloading or navigating away and unintentionally handing the lock to another open tab. 
* **Toast Notifications:** The lock-holding tab actively generates all notifications. Other open tabs mirror the persistent toasts, so you can watch progress from any window without those tabs executing tasks. You can restrict this behavior via `Settings` → `Notifications` → `Only notify on active tab or guild page`.

### Practical Notes

* **Automatic Handover:** Closing or reloading the active lock-holding tab hands the lock over to another open tab on the same account within a second or two. There is usually nothing you need to manage manually.
* **Navigation** the tab lock system is mostly for when you want to use multiple tabs and remain clutter-free of the script while its doing its work. If you use a single tab, the tab lock doesnt really matter and you can do so if you want, but I would recommend setting one tab as a "main tab".
  
---

## Smart attacking (why it "probes" first)

You'll notice the first couple of hits on a new monster type are weak Slash attacks. That's intentional — the script doesn't know your damage-per-stamina against that specific monster yet, so it throws a few cheap probe hits to measure it, then immediately switches to whichever skill clears the quest's minimum damage in the fewest stamina points. This avoids wasting stamina on overkill with a big skill and crits.

This calibration is *per monster type* and gets re-checked every few minutes (crits, gear, etc. can shift it), so don't be alarmed if you see a probe hit or two again later on a monster you've already fought.

---
## The Recheck Cycle: What Actually Happens While You're Waiting

When nothing usable exists yet for a quest—no spawned monster, no death confirmed, no loot ready—the script does not sit idle for the whole wait. It rechecks on a fixed timer (**Wave poll delay** in `Settings → Combat`, **10 seconds** by default), and the "waiting" toast you see counts down to that before rechecking. 

### Key Details About the Recheck Cycle

* **One Shared Timer**  
  It is one shared timer, not one per quest. If several quests are all waiting on a wave at once, you will see a single combined countdown. When it hits zero, the script runs one full decision pass.

* **Guild Quests Take Priority**  
  Every recheck tries guild quests first, before it even looks at Battle Pass (BP). That includes:
  * The quest that originally triggered the wait.
  * Anything else sitting in the same "waiting on a wave" state.
  * *(With Prekills on)* Any other quest with monsters it can hit right now.  
  If any of those have usable targets, the script attacks through them in order before moving on.

* **Battle Pass Is Sequential**  
  BP only gets a turn after the guild pass comes up empty. If guild work is genuinely unavailable on that cycle, the script does one BP cycle, then goes back to waiting for the next recheck.

* **Current Batches Finish First**  
  If the script is already processing a quest batch, it does not interrupt that batch for a new wave retry. For example, if we are currently attacking 7 eligible monsters, it will finish those 7 first. If monsters for another quest spawns during that time, it is not added to the active batch. It gets picked up on the next pass after the current batch ends.

* **"No runnable prekill targets"**  
  This does not mean the quests are done. It usually means they are attacked and now waiting on death or loot confirmation—with nothing new to attack at that exact moment.   This allows BP to fill during these moments where its only waiting.
---
## Potions: automatic stamina / HP / mana use

If **Use potions automatically** is on (Settings → Potions, or the toolbar checkbox), the script will:

- Use **small stamina potions** to top up before an attack if you don't have enough stamina for the hit it wants to land.
- Use a **full HP potion** if your HP hits 0 and you still have quests.
- Use **mana potions (S)** to keep skill-usage quests going if you're short on mana for the chosen skill.

Each potion type has a **per-run cap** in Settings → Potions (`-1` = unlimited, `0` = disabled entirely, any positive number = a hard limit for that run). Counts reset whenever a new report/run starts, and you can zero them early with **"Reset all counters now"** without touching the caps themselves, or creating a new snapshot. If a cap is hit and you genuinely run out, the script hard-stops with a "no stamina"/"no HP potion"/"no mana" message rather than silently failing.

---

## The status bar: badge, pause, and the lock system

Three small UI elements sit together at the bottom-left of the screen:

- **Status badge** — shows the current phase in plain words: e.g `IDLE`, `WAITING WAVE``ATTACKING` etc or hard stops — see troubleshooting. The dot pulses only on the tab actually doing the work.
- **⏸ / ▶ button** — pauses or resumes all automation. While paused, nothing runs, including Battle Pass filling and watchers.
- **🔒 button** — this is **not** the same as "which tab is doing the work." Only one tab at a time can hold the automation lock (handled automatically via the browser's lock API — whichever tab is open/active gets it, and it transfers to another open tab automatically on reload or navigation). The 🔒 button just adds a **"are you sure you want to leave?"** browser warning to that one tab, so you don't accidentally transfer the lock to another tab when you dont want it to. It's a guard rail, not the lock itself.

If you have multiple tabs open, only the lock-holding tab actually attacks/loots — other tabs mirror toast notifications so you can still see progress without duplicating actions. You can turn this off in settings > notifications if you want.

---

## Settings reference (by tab)

- **General** — theme switcher (Default / Void / Heavenly), the three master automation toggles (Autokills, Prekills, Skip Battle Pass, Loot Finder), display options (status badge, pause/lock buttons, tab-active pulse), where the status bar sits on screen, and the debug export button.
- **Quests** — cooldowns between repeated accept/donate/finish attempts, and the "promotion detection" tuning that decides how confident the script needs to be before wiping progress when your quest board fully changes (raise these if it ever resets your progress incorrectly during normal play).
- **Combat** — attack pacing, retry/backoff timing, the damage-calibration internals (how it learns your damage-per-stamina per monster), and manual recovery controls: **Clear hard stop** and **Full Reset**.
- **Potions** — the auto-use toggle, the "pause on no stamina" behavior, and per-run caps for each potion type.
- **Network** — HTTP retry counts/delays and cache lifetimes for things like wave scans, stamina reads, and Loot Finder's scan interval. Lower values mean more frequent server hits which could lead to issues as we get ratelimited
- **Notifications** — mute individual toast categories (quest events, kill progress, waiting/background toasts, errors, guild-verification pings), control toast durations, and tune how many past session reports/events are kept.
- **Waves** — turn built-in wave sources on/off, add extra wave URLs to scan, and manage the Battle Pass hunt target monster list described above. Most people never need to touch this — it's mainly there in case targets are on a wave that the script doesn't know about yet, or one of the built-in ones stops being useful for your quests (so you can skip scans for it).

Remember to save settings after you change them.

---

## Records (session reports)

The **View records** button opens a history of automation runs: total attacks, damage, kills, quests finished, loot claimed, and a full timeline of events you can filter by type or by quest. The current run is always "active" until it finishes or you manually snapshot it — **⚡ Snapshot** archives the current progress and starts a fresh report without interrupting automation. Old reports can be deleted individually.

---

## Special quest types

- **Skill-usage quests** ("use N skills") run automatically by spamming support skill on a random mob, but requires you to have **picked a class** at `/classes.php`. If you haven't, the script skips the quest with a "Skills Missing — No Class" toast and will do prekills on other quests (then just pick a class) - pause script if you dont want this.
- **Donation/collection quests** are recognized and automated for the donate/finish steps, but the script does **not** gather, craft, or buy items for you — you still need to physically have enough of the item. If you're short, it will do battle pass.

---

## If something goes wrong

1. Make sure you're on the tab that's actually doing the work (the one with the pulsing status dot / lock icon).
2. If the status badge says `STOPPED: ...`, that's a **hard stop**, not a crash — see the table below for what each one means. Hard stops clear themselves automatically after the recheck timer, or you can clear them immediately.
3. Hard Stops: Hit **⏸ to pause**, then go to **Settings → Combat → Clear hard stop**. This will clear any hard stops e.g no stamina timer that would of had retried in 60s.
4. **Settings → General → Export debug package**, and send the file to me in a DM. This is really useful for diagnosing issues + description of what happened.
5. As a last resort, **Settings → Combat → Full Reset** wipes all runtime state, kill queues, and pending loot. Please export a debug package *first* and send me a dm if you can — it's the only record of what went wrong.

### Hard stop reasons

| Badge says... | What happened | What to do |
|---|---|---|
| `STOPPED: no-stamina` | Ran out of stamina (or script predicted you don't have enough stamina to deal enough damage anymore), and potions are off (or you've hit your potion cap) | Wait for stamina to regen naturally, or check the cap in Settings → Potions |
| `STOPPED: no-hp-potion` | Hit 0 HP with no Full HP Potions left | Get more, or wait it out — it'll auto-retry every 60s |
| `STOPPED: no-mana` | A skill-usage quest needs mana and you're out of Mana Potions (S) | Same idea — restock or wait for the retry |
| `STOPPED: auto-farm-on` | The game's built-in Auto Farm is turned on | Turn Auto Farm off — it and this script can't run at the same time |

You usually don't need to manually clear anything for these — they retry on their own once the underlying issue is gone. Manual clearing is mainly for when you've fixed the cause and don't want to wait out the timer.

---

## Good to know

- This runs in your browser, not on a server — the tab holding the lock needs to not be closed (but if it gets closed, it will just transfer to any existing tab so no worries) for anything to keep happening.
- Saved quests, settings, and reports are stored locally in your browser and scoped to your account — and different accounts on the same browser stay separate.
