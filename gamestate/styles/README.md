# Styles
A Style is a collection of files that define how Components in your game are generated -- in other words, the look and feel of your vote counts, opening posts, role PMs, prods, etc. etc. All of these files are collected into a directory called a "style" so they can be grouped according to their aesthetic or the particular game they're intended for, and swapped out easily.

## Structure
Each style is a directory containing two types of file. The first is a "Component Body" file, which defines the physical structure of a Component. The easiest way to understand these is to look at the *\_default* style's vote_count component body (*vote_cout.component.body*):
```
{title}

{flavor}

{voters_list}

{not_voting_list}

With {active_voters} alive, it takes {threshold} to lynch.

{deadline}

{past_vote_counts}
```
Anything in curly braces is a placeholder for some data that will get filled in using Python's `.format()` method. These placeholders are called 'parts' of the Component within the code itself. A part can contain plain text or numbers, but can also contain other Components, called "subcomponents". It is entirely possible for a Component to contain a Component which contains multiple Components which contain Components!

The second type of file is a "Component Config" file, which contains a JSON dictionary with a list of default parameters for the Component. For example, the players_list component configs (*players_list.component.body*):
```json
{
	"include_replacements":true
}
```
Any players_list component generated under this style will be passed the parameter "True" for "include_replacements", meaning that all Players Lists generated will have replacements for each player included by default.

When creating a file, name it using this convention if it's a Component Body:
```
<component type>.component.body
```
And in this way if it's a Component Configs file:
```
<component type>.component.configs.json
```
If you do not include a body file for a particular component in your style directory, the default one (defined under the *"\_default"* style) will be used instead. There are also some default parameters defined in the configs files under the defaults. If you define any configs in your own style that contradict the default ones, yours will take precedence.

## Component Class
All components are subclasses of the Component class, and thus share a few methods in common. The most important is the `generate()` method, which tells the Component class to spit out its texual representation -- for instance, if it's a VoteCountComponent, it should produce a vote count. Whenever a Component is generated, it does the following.

1. Calls its `_validate()` method to make sure it was passed all of the parameters it needs.
2. Calls its `_get_parts()` method to get the values that will be plugged in to the placeholders in its body. If any placeholders have the same name as a Component type (e.g., "vote_count", which is the type of the VoteCountComponent), this method will automatically generate a subcomponent of that type to be plugged in. In addition, all Component types add their own logic based on the parts defined for that type.
3. Calls its `_body()` method to get its body. Most components pull this from their Component Body file, but there are some components, like the ListComponent, that override this method with a custom behavior.
4. Calls the `.format()` method against the body, with the parts generated in step 2 as parameters, to get the final result!

When you need to create a new component, use the static method `Component.create()` to do so, which takes a Component type as its first argument, and any number of parameters as its \*\*kwargs.

## Component Types
(under construction)

## Design Philosophy
* **Customization should be extremely simple.** When it comes to vote counts, role PMs, and OPs, there are as many sets of preferences as there are mods. The Component logic should be designed to allow mods to design their own styles without any Python or scripting knowledge whatsoever. The layout of the Component Body files should be as close to WYSIWYG as possible. Allow mods to ignore entirely parts of a Component they don't care about.
* **Make Components robust.** As few things about a Component as possible should be constrained; for instance, it should be possible to have the parts of a Component generate in an unorthodox order if desired. Parameters should exist to control all sorts of behavior about a Component, for mods who really want to go crazy designing their ideal style.
* **Encourage sharing and reuse.** Allow mods to share their styles easily by packaging everything in a single directory. Develop a number of "basic" styles usable easily by any mod.
* **It takes only a single call.** Make it possible to produce the most common types of components with a single call to the Game object.
* **Decouple style from state.** The state of a game isn't affected by the style in which its moderator posts are written, so keep all of the Component-specific logic out of the GameState and its related classes. Pass GameStates, Elections, etc. into Components so that mods can in theory generate the same vote count in various styles for the sake of comparison.
* **Components can see everything.** Allow any Component to fully inspect the GameState or related object passed to it. Components can look, but not touch.

## Future Plans
* Come up with a good way of allowing mods to construct standardized posts, like the OP, without having to make a separate component type for each.
* Add more methods to the Game class to make it easy to generate Components with a single call.
* Implement new Component types and new customization options within them.
* Create styles based on some of the most popular vote count and OP formats used by well-known mods, to make it easy for aspiring mods to emulate those styles.