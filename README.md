# Leaving Purradice 🐱🏝️
Very simple text game custom command for YAGPDB. Usable on free tier.

You are on an island with a bunch of cats. It is the chat's job to get the cats off the island.

Human written. AI assisted. I write this declaration for transparency. I don't blindly support AI.    
I am a purely self-taught "coder." I made this game for fun and to learn a bit.
<details>
<summary>How free-tier LLMs were used</summary>
I wrote most of the code myself. Free-tier LLMs (Mostly claude.ai but some ChatGPT) were used for assistance with the following:

**Brainstorming**    
- Helped predict common phrases and actions people might use during the game. Essentially a thesaurus.
- Gave suggestions on how to solve certain logical problems (did not write code)

**Regex**    
I wrote my own regex, but had AI optimize it.

**Troubleshooting & pointing out errors**    
I gave AI my files and told it to tell me any errors it saw. I then corrected the errors myself. The AI did not write code, but it did give suggestions.

**Rewriting particular sections of human-written code to not rely on regex**     
I did some of this myself, but basically any large block that relies on `in` was LLM-written. I had too many uses of `reFind`.

**Re-writing human-written code so the game could be played using the free-tier of YAGPDB**    
I self-host YAGPDB and have premium. But I wanted the game to be accessible to free-tier users. I did try to optimize it myself, but the hours I was putting in were not making sense for such a simple game. Eventually I gave the LLM all my files and told it to (essentially) fix it for me. And that is the version of the code that is in the repository.

The command was already split into 9 files by that point. The LLM added `react.yag` and made several changes to `actions.yag` and `main game.yag`.

Any changes after the initial repo commit are human-written.
</details>

## Start the game

[Install the custom commands](https://github.com/mothka/Leaving-Purradice#how-to-install-the-custom-commands), then have someone type `-startgame` in the channel.

A default game starts with 5 cats. If you want a different number, simply add that to the command.    
e.g. `-startgame 20` will start the game with 20 cats.

If a game is already running, only someone with manage message permissions can restart it.

## About the game mechanics

Discord users send messages. The game rolls a d100 and determines whether their actions will succeed or fail. Natural 1s and natural 100s cause debuffs and buffs for that user. 

Actions are determined via regex expressions. The message (in most cases) must start with a pronoun and contain a word for "cat". It also needs some type of pre-determined action verb.

**The game is so simple that you should not tell the users what the possible actions are. If you do, the fun is greatly diminished. Let them discover the commands and the method of game play on their own.**

Although a variety of actions respond with flavor text, the only actions that actually matter are trapping (and sometimes feeding) cats. The goal is to trap the cats on the island. You keep trapping until there are no cats left, at which point you win the game.
The game is co-operative, although there is a "release" action that brings cats back if someone wants to be cheeky.

**Passage of time**   
Every 100 messages or 30 minutes causes a new in-game day. Every 4 new days introduces 5 new cats. The game has no lose condition, but if you keep a stale game going it is certainly possible to get overrun with cats.

**Amount of cats to capture**    
The default game starts with 5, but with the `-startgame` command you can specify some other number.

**Interested cats**    
I wanted a 2-step process to make the game feel more deep than it was. The idea was you need to interest or attracts cats first before you can trap them. In practice, all this really affects is how many cats are trapped at one time. I ended up not doing much with this mechanic other than flavor text.    
It is possible to "disinterest" cats, but this is a fairly rare occurence.

**"Chatter" messages (non-actions)**    
The game rolls a d100 for every message. Most actions have somewhere around a 50% success rate.

If the message is not a command, it counts message characters. Messages under 50 characters with a roll of <=30 disinterest cats.   
If the character count and the roll is >=50, the chatter attracts (interests) more cats toward you.

**Rolls, buffs, and debuffs**    
Every message rolls a d100. Rolling 1 causes debuff "damage" while rolling a 100 causes a buff of random flavor. De/buffs last for 10 minutes or 10 messages, whichever is shorter.

Certain actions cause fake criticals no matter what the dice rolled. In most cases, this applies to buffs only, as a method of helping you to win the game easier.

Fake criticals do not stack. If you already have a de/buff, the larger de/buff wins and you get an extra 10 minutes/messages with it.

Real criticals (rolling a 1 or 100) stack. Your old de/buff gets added to the new one and the timer is reset.

De/buffs have a max cap of 50.

There is a set of actions in the game that can cause maximum damage every time. Those are the "1_nope" actions - overt violence toward cats. Doing one of these actions and rolling <= 50 causes a minimum of 20 damage and rewrites the roll as a 1, thus stacking any future damage while you have it.

### Possible Actions

Determined via regex. We call something a command when it contains a pronoun, one of the pre-determined verbs, and (in most cases) a word for "cat".

Each action is given a priority. If a command would match more than one, it goes to that of the highest priority, except in some special circumstances.

All except two regex are found in `actions.yag`.

**Perception check / "look"**    
This is in `main game.yag` and is the highest priority action.    
It searches for "look words". Additional words related to "help" or perception checks are also considered. If the message is 1-2 words it does not need a cat word or a pronoun to trigger.

It gives more information about the game in flavor text. It tells you how much time has passed, your equipment, the objective, how many cats are currently on the island, how many are interested in you, and how many you have trapped so far.

If you currently have a de/buff, there's some vague flavor text about whether you feel good or bad. The actual numbers are not mentioned.    
The entire response is also in `main game.yag`.

**Naming cats**    
In `main game.yag`, this looks for "name" words and allows you to name a cat. If the name used can be found via regex, the bot response uses it.    
e.g. Player says "I name the cat Steve." The response will be "Steve stares at you."     
Names are not persistent and the flavor text is lacking. Rolls <=40 the cat dislikes the name. >=70 the naming attracts more cats toward you.   
Functionally, it has no real bearing on game mechanics.     
Responses and the regexes for trying to find the name are in `naming.yag`.

**1_nope**    
Overt violence toward cats.    
Trying to hurt a cat in any way gives flavor text disparaging the player. Rolling <=50 rewrites the roll as a 1 and gives a minimum of 20 damage.

Because "throw" has ambiguous motivation, messages with the word are given special consideration and its own logic branch, which looks for other words to help determine what action it actually is. Possible ones are feeding (3_call), trapping (6_grab), or playing (4_pet).    
Responses for this action are in `react.yag`.

**2_drugs**    
If you try to tranquilize or spay/neuter the cats, this simply responds with not having the right equipment for that.    
Responses for this action are in `react.yag`.

**3_call**    
Essentially any words that could possibly attract and interest more cats.    
With the presence of a food word, in certain situations this turns into a trapping action instead. Responses for this action are in `react.yag`.

**4_pet**    
You can have fun with the cat. Show affection, pet, play, take a photo. Photographs have the best flavor text. These actions usually make more cats interested in you, but are not that important mechanically. Responses for this action are in `petting.yag`.

**5_intimidate**    
Being slightly mean toward the cats but not violent. Running, chasing, yelling, trying to scare them. Has a chance of disinteresting cats on successful rolls.    
Responses for this action are in `react.yag`.

**6_grab**    
This is a player trying to run and grab a cat, essentially. If they are trying to capture a cat without specifying how, or without using the word "trap," it goes to this action. The maximum amount of cats that can be caught this way is 1, and it happens on rolls >=90.    
Responses for this action are in `react.yag`.

**7_free**    
Any attempt to release previously-trapped cats. Doesn't work if you haven't caught any cats yet. Otherwise, you free a random amount of the trapped cats with a >= 70 roll.

Freed cats subtract from your total caught cat count and return them to the total cats on the island. You must trap them again to finish the game.    
Responses for this action are in `release.yag`.

**Feeding & trapping**    
If you mention food in your message, we try to keep that word and refer to it later. Throwing food counts as 3_call. Using the word "bait" or "trap" counts as trapping. 

Originally setting a trap and getting cats were supposed to be separate actions, but as the file count ballooned I combined them into one response. 

If you set a trap of any kind, we resolve the entire process, which is found in `trap_feed.yag`:

- traps without a food word get a 20 point debuff to the roll
- \>=50 roll will catch already-interested cats
- If all cats on the island are interested and you didn't roll a 1, you still catch at least 1 cat
- If you rolled >=40 and there are cats that aren't interested yet, the traps attract even more cats toward you.
  - The amount doubles and is treated as a 100 roll if you rolled >=90.

## About YAGPDB

Just in case you got here randomly, [YAGPDB](https://github.com/botlabs-gg/yagpdb) is an open-source discord bot. This game uses the custom command system of this bot. It can run on the free tier of the public instance.

Since YAGPDB custom commands do not use a real coding language, this game won't work without the bot and probably cannot be easily translated.

# How to install the custom commands

1. Make a new custom command group named "Leaving Purradice."
2. Make sure this custom command group only runs in one channel.
3. Create a new custom command for each file and name the command the filename. Make sure all the commands are in the Leaving Purradice command group.
4. Add all commands *except* `main game.yag`. ALL commands have a trigger type of "none".
5. Add `main game.yag` last. It has a trigger type of `regex`. The regex trigger is `\A`.
6. Inside the command, under `{{/* CCIDs */}}`, change all the variables so the number refers to that custom command's ID in your control panel.    
For example, `$startgame` should go to the command you made for `startgame.yag`.

CC IDs are the number at the top left on the control panel. E.g. `#32 - None:` is `32`. So in our example, you would put `{{$startgame := 32}}` in our `main game` command.

7. **Optional: running this game in a thread.**    
To run the game in a thread, set the `Leaving Purradice` command group to only run in the parent channel.

In Discord, create your thread. Then copy the thread's ID.    
At the top of the `main game` command, add this code:    
```
{{if ne .Channel.ID 1234}}
	{{return}}
{{end}}
```

Replace `1234` with your thread ID.

# Licensing & Contribution

Use it however you like. You do not have to credit me. Modify it as you like. I would love it if you contribute to the repo, but I will not approve pull requests if they would make the game no longer playable on the free tier of YAGPDB.

I am unlikely to fix issues for this game.

# Known Issues

- To get around `execCC` stack depth limit, I save information to a database and retrieve it from the other commands. This causes 1-6 seconds of lag. In the rare case the data isn't found within 6 seconds, the trigger message simply isn't recognized.
- There's no way to end the game early without catching the cats or starting a new game. This is ultimately harmless; only one database doesn't expire.
- All die roll success/fails were pretty much pulled out of my butt. The game has been played a few times, but it is not fully play-tested. Some odds may feel very out of balance.