# Events
Events are things that occur that can alter the state of the game. They are intended to be the sole means by which the GameState is modified -- no direct alteration of the GameState, even by method calls, should be carried out. The use of Events in this way ensures that changes to the GameState occur in a well-defined and predictable way, reducing the chance of mod error.

The Game object contains a list of Events, which can be loaded in through a couple of methods. When the Game object applies the Event to the GameState, it's said to be *processing* the Event. An Event is processed by calling its `execute()` method, with the GameState it's being processed against passed in as a parameter.

## Structure
Events are represented by JSON objects in a form resembling the following:
```json
{
	"type":"vote",
	"post":100,
	"voter":"GreenLiquid",
	"votee":"Radiant Cowbells"
}
```
The current design of the Game object allows events to be loaded en masse from a .JSON file containing a list of Event JSON objects. When you are creating your file, name it after the name of your Game like so:
```
<game name>.events.json
```
Every event must contain two particular keys:
* **Type:** the type of an Event tells the code which Event subclass to use. Whatever is supplied here must match the "TYPE" constant of an Event subclass.
* **Post:** indicates when the Event occurred. It is certainly possible to have multiple events occur on the same post -- for example, having Day end (a 'phase_change' Event) and someone die (a 'death' Event) on the same post happens all the time as a result of a lynch.

Within the events.json file, your events do not need to be listed in chronological order. The Game class contains logic to sort events when they are loaded.

## Types
The types of Events currently implemented are listed below. Note that I omit "type" and "post" from the Required Parameters, which are mandatory for all Events.

### VoteEvent
Represents an elector voting for another elector in an election; in most circumstances, this is a player voting to lynch another player (or No Lynch).

**Required Parameters:**
* voter: the name of the elector casting the vote.
* votee: the name of the player being voted for.

**Optional Parameters:**
* election_type: the type of the election the vote is intended for; useful when there are multiple elections.
* election_name: the name of the election the vote is intended for; also useful for multiple elections.

### UnvoteEvent
Represents an elector rescinding their current vote in an election.

**Required Parameters:**
* voter: the name of the player who is rescinding their vote.

**Optional Parameters:**
* votee: the name of the player who is no longer being voted; handy if multiple simultaneous votes are allowed.
* election_type: the type of the election the vote is intended for; useful when there are multiple elections.
* election_name: the name of the election the vote is intended for; also useful for multiple elections.

### DeathEvent
Represents a player dying for whatever reason (night kill, modkill, lynch, etc.).

**Required Parameters:**
* deceased: the name of the player kicking the bucket

**Optional Parameters:**
* flavor: the death flavor, which should be written to replace the word 'died' in the player's flip

### DeadlineEvent
Represents a deadline being set or adjusted, for the active phase, election(s), or in general. The deadline is presumed to apply to active elections if any exist; otherwise, it will apply to the active phase.

**Required Parameters:**
* deadline: the deadline being set, in ISO 8601 format.

**Optional Parameters:**
* election: if supplied, the deadline will apply to the supplied election only.
* phase: if set True, the deadline will apply to the active phase even if there are active elections.

### VoteCountEvent
Represents a vote count being generated and posted for an election. This is used to allow vote counts to link back to old vote counts, or for the OP to contain links to vote counts.

**Required Parameters:**
None

**Optional Parameters:**
* election_type: the type of the election getting a vote count; useful when there are multiple elections.
* election_name: the name of the election getting a vote count; also useful for multiple elections.

### PhaseChangeEvent
Represents the game advancing to a new phase, such as day transitioning to night or vice-versa

**Required Parameters:**
None

**Optional Parameters:**
* phase: the name of the phase to advance to. If omitted, the game's phase_progression modifier will be consulted to determine the next phase.

### ReplacementEvent
Represents a player replacing another player.

**Required Parameters:**
* replacee: the name of the player who is being replaced.
* replacement_name: the name of the player who is replacing in.

**Optional Parameters:**
* aliases: the aliases of the player who is replacing in (such as hydra heads).

## Design Philosophy
* **Events always have timestamps.** Events are initiated by posts, PMs, or logic in the GameState or Game object, all three of which are bound by time. Therefore, all Events should be marked with a timestamp or post number to indicate when they occurred.
* **Events must be processed in order.** There is currently no facility within the Game logic to process Events out of chronological order. If an Event is received which pre-dates an already-processed Event, all Events in the Game must be reprocessed from the beginning.
* **Events can be mistaken.** An Event may turn out to have no effect on the state of the game. It represents only what a moderator or modbot *thinks* is being attempted. For instance, it is completely valid for a 'vote' Event to be submitted against a player who cannot be voted. It is up to the logic within the Event and GameState to ensure this does not not actually cause a Vote to be created. However, some types of Event should never be mistaken, especially moderator-initiated ones such as 'vote_count' Events -- if such an Event is mistaken, an exception should be raised.
* **Process first, then parse.** The Game object should not initiate parsing of a post or PM until all Events preceding it have been processed. This allowes parsing to take advantage of information contained within the GameState without the risk of being out-of-date. For example, suppose a user replaces in and then in the next post is immediately voted by name. If the Game tried to *parse* both events before *processing* the first, it wouldn't know that the user named in the Vote is a valid User.
* **Events should not procede proof.** An Event should not be generated until there is a post, PM, or other article that demonstrates it has occurred. For example, if the ModBot receives a PM from a user confirming that they will replace someone in the game, the ReplacementEvent should not be generated until it has made a post in the game thread confirming this to be the case. This prevents posting race conditions where the ModBot considers an Event already binding on the GameState right before a player posts something invalidating it.

## Future Plans
If the codebase is ever converted or expanded upon to facilitate the complete automation of games (or automation of particular moderator functions, at least), Events will need to occur based on exact timestamps instead of post numbers, so that Events not taking place in the same thread can be evaluated for execution order.

Currently I have a nasty 'PRIORITY' constant in place for each type of Event that tells the Game object which takes precedence in the event that both occur in the same post. This is problematic for many reasons, one of which is that it's not possible to both start Day and post a vote count in the same post (the game will attempt to post the vote count for the Night phase). If the system is completely automated, this issue can be largely overcome by having the ModBot not post events that conflict within the same post, but there will still be issues with, for example, actions that will require some kind of extra logic.