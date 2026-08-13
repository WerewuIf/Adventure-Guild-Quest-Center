# Adventure Guild Quest Center

> Automates your Adventurer's Guild quests end to end: accepting quests, fighting, looting, donating, and finishing them while you do other things. It also supports Battle Pass hunts and the game's Auto Farm.

This guide covers setup, the important options, and the less obvious behavior that is useful to know while it runs.

---

## Features

* Automatic accept → combat → loot/donate → finish
* **Autokills** for normal kill quests
* **Prekills** for multiple kill quests concurrently
* **Loot Finder** for qualifying kills made elsewhere
* **Battle Pass** hunt automation and gap filling
* **Auto Farm** integration, including capture/restore

---

## Requirements

* [Tampermonkey](https://www.tampermonkey.net/)
* **[AHAB] Casual Ahab** with the **[AHAB]** tag in your display name

The script checks guild authorization on load and will block automation if the account is not verified.

---

## Installation

1. Install Tampermonkey.
2. Create a new userscript.
3. Paste the entire script and save.
4. Open/reload Demonicscans.

The controls are added to the **Adventurer's Guild** page.

---

## Getting started

1. Open the **Adventurer's Guild**.
2. Select the quests you want automated.
3. Click **Add selected**.
4. Turn on **Autokills** if you want automatic combat.
5. Turn on **Prekills** if you want multiple kill quests worked on together.
6. Choose **Manual** or **Autofarm** combat mode.
7. Turn on **Potions** if you want automatic potion use.
8. Unpause the script if needed.


## Autokills

Autokills is the main combat switch.

When enabled, the script finds valid monsters, joins, attacks, waits for death/loot, and continues until the quest is complete.

Without Autokills, quest actions such as accept, donate, and finish can still run, but the script will not perform normal monster combat.

**Prekills requires Autokills.**

---

## Prekills

Prekills lets the script work on multiple kill quests at the same time instead of finishing one before starting another.

The main advantage is less downtime while one quest is waiting on waves, deaths, or loot.

Kills can also be tracked before their quest is accepted. When that quest becomes active, the deferred loot can be claimed.

For several kill quests, **Prekills is the recommended setting**.

---

## Smart attacking

The first hits on a new monster type may be cheap probe attacks. The script uses them to estimate damage per stamina and pick a better attack for the quest's minimum damage requirement.

The estimate expires after the configured damage-model lifetime, so it can be recalculated later if your damage changes.

---

## Wave sources

The built-in wave sources are currently:

* Gate 3 / Wave 3
* Gate 3 / Wave 5
* Gate 3 / Wave 8
* Gate 5 / Wave 9

Go to **Settings → Waves & Potions → Wave Sources** to add custom sources or disable built-in ones.

Custom sources are mainly useful when a quest monster is on a wave the default list does not include.

The script also normalizes monster names when matching targets, so small naming differences generally do not break detection.

---

## Battle Pass

The script supports the **Battle Pass Hunt** separately from guild quests.

**Default hunt targets**

* **Lizardman Flamecaster**
* **Lizardman Shadowclaw**

Change them under **Settings → Waves & Potions → Battle Pass Targets**.

**Gap filling**

In Manual mode, Battle Pass hunt work can fill periods where guild automation has nothing useful to do.

The priority is always:

**Guild work first → Battle Pass only when guild work is unavailable.**

As soon as guild work becomes available again, it gets priority.

**Skip Battle Pass**

**Settings → General → Skip Battle Pass**

Enable this if you do not want the script to touch Battle Pass automation.

Battle Pass stamina progress is reported, but is not directly farmed by the script.

---

## Autofarm

Combat Mode has two choices:

* **Manual**
* **Autofarm**

Autofarm mode sends the current quest workload to the game's Auto Farm system instead of having the script perform the attacks itself.

It can configure the target monster, minimum damage, stack limit, total kills, and related Auto Farm settings, then monitor the live Auto Farm state.

**Multiple quests**

The current Auto Farm system **merges targets**.

Starting another quest does not automatically remove a target that is already being farmed by the script. Multiple quest targets can be in the same Auto Farm batch.

**Battle Pass**

Use Auto Farm for Battle Pass hunts toocontrols whether the Battle Pass Hunt is added to those batches.

**Capture / restore**

The Autofarm panel has:

* **Capture current config**
* **Restore captured config**
* **Clear captured config**

With **Auto-capture & restore** enabled, the script saves your current Auto Farm setup before taking it over and restores it after its automation work is finished.


**Potion synchronization**

Auto Farm can will get overwritten by these settings, and restored once over.

---

## Loot Finder

Loot Finder is independent of Autokills.

It looks for dead monsters that match an enabled kill quest, already have enough of your damage to satisfy its minimum-damage requirement, and still have loot available.

This is useful for kills made by something else, such as manual attacks or another bot.

It can:

* credit and loot active quests,
* defer kills for quests that are not accepted yet,
* and process stacked monsters.

Singles are preferred before stacks, with an overshoot limit (1.5x) to avoid consuming an unnecessarily large stack for a small quest.

---

## Quest types

**Kill quests**

Handled automatically when Autokills is enabled.

**Collection / donation quests**

The script can handle the donate/finish part, but you still need to have the required items.

**Skill-usage quests**

Quests such as **Use N skills** are supported.

The script checks your class, uses the mapped class skill, finds a target, handles the required mana, and tracks the skill uses.

You must have a class selected at `/classes.php`.

Current class mappings include Warrior, Mage, Hunter, and Cleric.

---

## Potions

The toolbar **Potions** toggle enables automatic potion use.

Supported:

* **Small Stamina Potion**
* **Full HP Potion**
* **Mana Potion S**

Stamina potions can be used when an attack needs more stamina than you have. HP potions can recover you after reaching 0 HP. Mana Potion S can keep skill-usage quests moving.

**Per-run limits**

* `-1` = unlimited
* `0` = disabled
* positive number = maximum for that run

**Reset all counters now** only resets the used counters; it does not change the configured limits.

---

## Waiting / rechecks

When the script is waiting for a wave, death confirmation, loot, or another temporary condition, it uses retry/recheck timers rather than repeatedly hammering the same action.

The relevant timing controls are under **Settings → Combat**.

When several quests are waiting, they are handled through the same automation scheduler.

---

## Multi-tab behavior

Only one tab for an account performs the actual automation actions:

* attacks
* loot
* accept
* donate
* finish

Other tabs can remain open and mirror notifications without performing duplicate actions.

The automation lease is account-scoped, and another eligible tab can take over when the current worker reloads, closes, or navigates away.

**The lock button**

The **🔒** button is a navigation guard for the current worker tab.

It is **not** a manual worker selector. The automation lease itself is handled automatically.

**Notifications**

Other tabs can mirror the worker's persistent toasts.

You can restrict this under **Settings → Notifications → Lock Notifications to Guild + Active tab**.

---

## Status bar

The bottom-left status area shows the current automation state and provides pause/control buttons.

Typical states include:

`IDLE` · `ATTACKING` · `LOOTING` · `WAITING WAVE` · `WAITING DEATH` · `AUTOFARM RUNNING ` · `PAUSED`

The worker tab is indicated by the status pulse.

---

## Settings

The available settings are grouped into the following areas:

| Settings area       | What it controls                                                                                                                                                                        |
| ------------------- | --------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| **General**         | Themes, Autokills, Prekills, Skip Battle Pass, Loot Finder, status controls, and debug export. Themes: **Default**, **Void**, **Heavenly**.                                             |
| **Quests**          | Quest action timing and quest-board change detection.                                                                                                                                   |
| **Combat**          | Attack timing, probe hits, target delay, death/loot polling, damage-model lifetime, wave polling, retries, and recovery controls. Also contains **Clear hard stop** and **Full Reset**. |
| **Waves & Potions** | Wave sources, Battle Pass targets, potion settings/caps, and Autofarm controls when Autofarm mode is selected.                                                                          |
| **Notifications**   | Toast filters, duration, and active-tab notification behavior. ---                                                                                                                      |

## Records

**View records** opens the current session report and past archived reports.

Reports track useful totals and events such as attacks, damage, loot, donations, completions, potion use, errors, and Battle Pass activity.

Use the report when checking what the automation actually did rather than relying on the last toast on screen.

---

## Hard stops / recovery

Hard stops are safety states that prevent the same failed action from being retried indefinitely.

| Hard stop               | Meaning                                                                             |
| ----------------------- | ----------------------------------------------------------------------------------- |
| `STOPPED: no-stamina`   | Not enough stamina for the intended action and it could not currently be recovered. |
| `STOPPED: no-hp-potion` | HP reached 0 and no usable Full HP Potion is available.                             |
| `STOPPED: no-mana`      | A skill quest needs mana and the script cannot refill it.                           |
| `STOPPED: auto-farm-on` | Manual combat is being blocked because the game's Auto Farm is enabled.             |

In Manual mode, the current build also has recovery for a stray game-side Auto Farm session.

**Clear hard stop**

Use this after fixing the cause if you do not want to wait for the normal retry.

**Full Reset**

Use only when the automation is genuinely stuck. It clears runtime state, kill queues, and pending/deferred work.

**Export a debug package first.**

## Troubleshooting

**Nothing attacks**

Check that:

* Autokills is on
* the quest was added with **Add selected**
* the monster exists on a configured wave
* the script is not stopped
* Manual mode is not being blocked by the game's Auto Farm
* the worker tab is still available
* you have enough stamina / usable potions

**Skill quest is stuck**

Check that you have a class selected at `/classes.php` and enough mana / Mana Potion S.

**Collection quest is stuck**

Check your inventory. The script does not gather missing quest items.


**Multiple tabs look confusing**

Remember: **one tab works, the others watch.** Reloading or navigating away from the worker can transfer the automation lease to another open tab.

---

## Debugging

Use **Settings → General → Export debug package**.

The exported `.txt` contains the runtime/configuration information needed to diagnose automation issues, including reports/events and Auto Farm state.

When reporting a bug, include the export plus a short description of what should have happened, what actually happened, and the quest/monster involved.

---

## Good to know

* The worker tab must remain available for automation to continue.
* Opening more tabs does not create more workers.
* Quest/config/report state is stored locally in the browser and scoped to the account where applicable.
* Avoid aggressively lowering timing values; it increases request frequency and can make rate limiting more likely.
