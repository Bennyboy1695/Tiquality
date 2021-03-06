# Tiquality
[Tiquality](https://minecraft.curseforge.com/projects/tiquality) is the successor of the seemingly popular mod: LagGoggles.

At this moment, Tiquality is a serverside only mod, but supports clients too (Lan hosted).

The core functionality is limiting a player's tick time in the world to evenly distribute time.

## Download
You can download the latest release [here](https://minecraft.curseforge.com/projects/tiquality/files)

## Modpack authors
Yes, you have my permission! Please add this to your modpack! :thumbsup: It will save server admins alot of time if you do all the configuration work for them! (`/tq add <feet|below>`)

## What this is, and what it is intended for
An update throttler, that aims to provide a fair Minecraft experience at 20 TPS for users who do not cause a heavy load on the server, whilst players who build carelessly only get 10 TPS (depending on their impact).

This encourages server friendly build behavior, and attempts to not ruin gameplay for others who care about their server.

## What this is not
 * This is not something like [ClearLagg](https://dev.bukkit.org/projects/clearlagg). Entities like sheep, dropped items, etc are NOT limited or removed <sup>1</sup>. Tiquality tries to keep the game fair for everyone, regardless of what someone else is doing.
 * This will not fix lag caused by huge mobfarms because of too many entities, entities (except TileEntities) are left untouched. <sup>1</sup>

<sup>1</sup>: If you have GriefPrevention installed, entities inside claims are limited aswell. Players are _never_ limited.

## Behavior
Blocks without an owner which are not specified in the config cannot update. This includes freshly generated areas. To keep functionality like leaf decay and grass growth, you can **whitelist** blocks in the config. Whitelisted blocks do not need an owner in order to tick.

If a player places a block, Tiquality will assign that block to that player's personal tracker. This tracker will perform future updates for that block, using the time constraint of the player that placed the block.

If more players log in, the time will be divided more. If a player doesn't use up all of his tick time, other's will receive the remaining time.

Logged out player's blocks have less tick time than players who are online (Config customizable)!

## WARNING
Removing Tiquality will **PERMANENTLY** remove it's existing data on chunk load on a per-chunk basis.

If you are looking to permanently delete Tiquality, simply start the server without it, and all data will get lost over time.

## How it functions
All calls to `Block.randomTickBlock()`, `Block.updateTickBlock()` and `ITickable.tickTileEntity()` are redirected to Tiquality, which in turn finds the owner of a block using a customized high performance lookup.

There are trackers associated with the block positions of the world, these trackers record how long an update took, and substract that from the 'granted nanoseconds' until it reaches zero. When it reaches zero all future updates are queued for later, and doesn't tick right away.

When the next tick comes around, all trackers get a granted amount of tracking time, meaning blocks update again. The queue is executed first. If the queue doesn't complete, new updates will be added to the queue if it isn't already.

## Commands and permissions
 * /tiquality
   * Main command
   * tiquality.use
   
 * /tq
   * Alias for /tiquality
   * tiquality.use
   
 * /tq info [point]
   * Prints info about the block you're standing on and the block your feet are inside (Liquids).<br>Prints the owner of the block(s)<br>Prints the current status if a block is on the whitelist or not.<br>If used with the 'point' flag, you can aim at blocks and sneak for quick info on who tracked a block.
   * tiquality.use
  
 * /tq track
   * Claims a block for a player without actually having to re-place it. (Things found in nature, existing bases... etc)
   * tiquality.use
   
 * /tq profile &lt;seconds&gt;
   * Runs a very basic profiler on blocks that you own. A better alternative is in the works.
   * tiquality.use
  
 * /tq profile &lt;seconds&gt; &lt;uuid or playername&gt;
   * Runs a very basic profiler on the targeted UUID or playername. A better alternative is in the works.
   * tiquality.admin
 
 * /tq add <feet|below>
   * Adds a block to the whitelist, making blocks of that type tick without owners.
   * tiquality.admin
  
 * /tq reload
   * Reloads the config file.
   * tiquality.admin
   
 * /tq import_griefprevention
   * Will update every GriefPrevention claim existing before Tiquality was installed with TiqualityTrackers. You only need to run this once during first run! This command is hidden from TAB autocompletion.
   * tiquality.admin
   
 * /tq import_griefprevention_claim
   * Will update the GriefPrevention claim (where you are currently standing) existing before Tiquality was installed with TiqualityTrackers. You must use this command if the command above loaded a corrupt chunk you do not intend to fix. You only need to run this once for each claim that existed before Tiquality was installed! This command is hidden from TAB autocompletion. Abuse is prevented by only updating if no tracker yet exists.
   * tiquality.use

## Frequently asked questions
 
- [Why don't you move to Sponge already!?](#why-dont-you-move-to-sponge-already)
- [What does `/tq info` do?](#what-does-tq-info-do)
- [My blocks don't tick! What do I do?](#my-blocks-dont-tick-what-do-i-do)
- [My fluids don't flow! What do I do?](#my-fluids-dont-flow-what-do-i-do)
- [What is your code style?](#what-is-your-code-style)
- [I just installed Tiquality, and the TPS is HORRIBLE!](#i-just-installed-tiquality-and-the-tps-is-horrible)
- [Why store data in the chunk itself?](#why-store-data-in-the-chunk-itself)
- [You suck!](#you-suck)

### Why don't you move to Sponge already!?
It is my intention to make Tiquality as widely available to everyone. Not having to install Sponge, match the Forge version, find mods that are both compatible with Sponge and that specific Forge version makes it easier to install. Everyone should be able to use Tiquality, *even if you run a Sponge-free server.*

**Sponge is still supported, however.**

### What does `/tq info` do?
It helps you diagnose if a block can be ticked or not

Stand on top of the block and use `/tq info`. The output will be as follows:
```
Block below: minecraft:piston tracked only Status: Tracked by: Terminator_NL
```
We can break this down:
 - "minecraft:piston" is the block name
 - "tracked only" means that this block will only tick if a tracker has been assigned (See: [/tq track](#commands-and-permissions))
 - "Tracked by : XXX" tells us if a block is tracked or not, in this case: It is tracked by me.
 
Another example of `tq info`:
```
Block below: minecraft:sand whitelisted Status: Not tracked
```
We can break this down:
 - "minecraft:sand" is the block name
 - "whitelisted" means that this block **type** will tick, regardless of it being tracked or not.
 - "Not tracked" tells us that no tracker has been assigned to this block.

Recommended usage:
 - `tq info point` which allows you to see what blocks are tracked simply by looking at them, and sneaking.
 - `tq info` to see if your config changes worked

### My blocks don't tick! What do I do?

A block will tick if at least one of the following statements is true:
 - There's a Tracker assigned and the tracker has enough time to tick the block
 - The block is defined in the config (`AUTO_WORLD_ASSIGNED_OBJECTS`) and the tracker has enough time to tick the block
 - The block is defined in the config (`TICKFORCING`) It will tick even if a tracker has been assigned that ran out of time.
 
The fastest way to solve this is simply by standing on the block and running `/tq add below`. It will add the block to the config under `AUTO_WORLD_ASSIGNED_OBJECTS`.

Protip: Use [`/tq info`](#what-does-tq-info-do) first, to see if you are actually positioned on the block correctly.

### My fluids don't flow! What do I do?

Fluid's are tracked the same way as [blocks](#my-blocks-dont-tick-what-do-i-do).


The fastest way to solve this is simply by standing in the liquid and running `/tq add feet`. It will add the fluid to the config under `AUTO_WORLD_ASSIGNED_OBJECTS`.

Protip: Use [`/tq info`](#what-does-tq-info-do) first, to see if you are actually positioned in the liquid correctly.


### I just installed Tiquality, and the TPS is HORRIBLE!
This is because blocks are trying to tick, but have no tracker assigned. If you don't have GriefPrevention installed, you have to reset your world to fully enjoy Tiquality.

If you do have GriefPrevention installed: try to use the `/tq import_griefprevention` command. Be wary, as most servers contain corrupted chunks, and this command will load them.
If you can't import because you have corrupted chunks, use: `/tq import_griefprevention_claim`. It will only import the claim you are currently standing in, but after everyone runs that command everything should be running smooth.

More alternatives to GriefPrevention are on the to-do list.

---

Keep in mind, that there are MANY other reasons the TPS can still not be 20. Chunkloading unloaded chunks is one of the big effectors: people exploring, or logging in. Another big effector is post-world-tick processing.
Tiquality does **NOT** hook into world events, so any processing AFTER the world tick is completely outside of Tiquality's control.


### Why store data in the chunk itself?
Storing data in the chunk allows for easier resets and makes sure the data does not go out of sync with the chunk.
Inspired by minecraft's own code, Tiquality also uses bitshifting to find the right identifier for a block, without having to iterate on anything. This means blazing fast performance, and this is needed because Tiquality has to **intercept and act on every ticked object**.

### You suck!
Hey, I am just trying to make the world a better place, I am sorry it did not work out for you.
At the time of writing I have had serveral encounters where people flat out accuse me of doing everything wrong, and simply put: It hurts, lets not do this?

### What is your code style?

I follow my own set of rules whilst coding, to keep intent as clear as possible.
 - `== true` clarifies intent
 - `== false` is much more visible than `!`
 - There's no shorthands like `if()return;` I **always** use blocks
  ```
  if(){
     return;
  }
  ```
 - There are no lambdas, simply because I don't like them for readability purposes (Subject to change)
 - AtomicInteger can double as object for synchronization signaling when waiting for *N* tasks to end, instead of sleeping for a set time.

If you feel like you found something that needs to change, please follow the rules above before submitting a pull request.
Another note: Please notify me beforehand if you intent to drop a big pull request, so I can give you some feedback if it will make it in the master branch before you waste alot of time on something I already considered.