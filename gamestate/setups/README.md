# Setups
There is no class in the codebase (yet) to represent a setup, but the files in this directory are still invaluable for initializing the list of players in a game, setting their roles and alignments, and defining PhaseTypes and ElectionTypes for use by the game.

## Structure
Each setup is a JSON object in a form resembling the following:
```json
{
	"name":"Mini 2200: GreenLiquid's Overwhelmingly Bastard Game",
	"type":"Mini Theme",
	"mods":
	[
		{
			"name":"GreenLiquid",
			"aliases":[]
		}
	],
	"players":
	[
		{
			"name":"Radiant Cowbells",
			"aliases":[],
			"role":
			{
				"flavor_name":"Regis Philbin",
				"role_name":"Mafia Godfather",
				"abilities":[]
			},
			"alignment":"Mafia",
			"modifiers":[]
		}
	],
	"phase_types":
	[
		{
			"name":"Day",
			"elections":["Lynch"],
			"modifiers":{}
		}
	],
	"election_types":
	[
		{
			"name":"Lynch",
			"modifiers":
			{
				"include_all_players":true,
				"end_on_phase_change":true
			}
		}
	],
	"modifiers":
	{
		"phase_progression":["Day", "Night"]
	}
}
```
Let's break it down by the top-level keys:
* **Name:** the name of the game. Not currently used, but could be eventually for setting the title of the thread and figuring out which PMs in the mod's inbox are actually intended for the game.
* **Type:** the type of game, according to MafiaScum's queueing system. Also currently unused, but in the future in could be handy for determining the correct subforum for posting the game thread.
* **Players:** a list of all players in the game along with their name, role, alignment, and starting modifiers. The "name" and "aliases" are used to generate the starting User for the Player object. Currently the info under "role" is not used in any way. Eventually, the "abilities" could be used to automate power role resolution.
* **Phase Types:** a list of PhaseTypes used by this game. A PhaseType is a template for Phases that have the same naming convention and modifiers in common, such as Day and Night, and are used to generate Phase objects.
* **Election Types:** a list of ElectionTypes used by this game. An ElectionType is a template for Elections that have the same modifiers in common.
* **Modifiers:** a dictionary of modifiers that apply to the entire GameState. Notably, the "phase_progression" modifier tells the GameState which Phase to advance to when a PhaseChangeEvent occurs without a phase specified. These loop indefinitely. By default it's [Day, Night].

When you are creating your setup file, name it after the name of your Game like so:
```
<game name>.setup.json
```
There is an already-existing setup file called "\_defaults.setup.json" that is loaded first, prior to your game's setup file. It defines the following:
* **Phase Types:** Day, Night, Pregame, and Endgame
* **Election Types:** Lynch
* **Modifiers:** "phase_progression":["Day","Night"] *(in other words, day start assumed)*

You do not need to specify any of these types or modifiers in your own setup file unless you need to redefine them in some way. If a type/modifier exists in the \_default, but not in your setup file, the version from the \_default will be used; if one exists in both places, the version in your setup file takes precedence.

## Design Philosophy
* **Make normal games easy.** If a mod wants to run a normal game, they should have to supply nothing more than their game's name and type, their own User info, and their player/role list. The default modifiers and types should work perfectly for a normal game of Mafia, with only small tweaks to represent things like plurality lynching needed.
* **Extensibility pays off later.** Modifiers can be included for the GameState itself, all Players within it, all PhaseTypes, and all ElectionTypes. This allows for custom functionality to be added later and utilized only in cases where it's explicitly called for.

## Future Plans
* Make the role info actually do something.
* Determine if it makes sense to put the thread URL / PT URLs in here -- currently, I'm thinking 'no' because that information is really only needed if the ModBot is automating everything, and in that case it would be expected to make the thread and any PTs itself.