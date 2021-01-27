# Mych's Macro Magic

A simple, sane, and friendly little scripting language for your Roll20 macros.

- Tired of writing macros that look like the cat had an uneasy dream (again!) on your keyboard?
- Stockholm syndrome just not working for you to keep you enchanted with that soup of brackets, braces, and strings of seemingly random dice-roll suffixes you wrote yesterday?
- Your [sense of pride and accomplishment](https://www.reddit.com/r/MuseumOfReddit/comments/8ish3t/the_time_that_ea_got_a_sense_of_pride_and/) just not kicking in anymore like it used to after spending hours figuring out how to bully the dice engine into comparing two numbers?

And that distant, lingering, but unmistakable yearning, like an unscratchable itch at the back of your mind, calling softly at you as if from afar: "How easy would this be if I could just use a proper *conditional* here?", and: "How nice if I didn't have to put all this stuff into *one* line?"...

If that sounds familiar, then **Mych's Macro Magic (MMM)** is here for you. Just install it as an API script and start writing macro scripts like a boring, *productive* programmer would.

You're here to yell at dice, not at macros, after all – right?


## Scripts

You can write MMM scripts wherever you can write macro code: in the *Macros* tab in the sidebar, in your character sheet's *Abilities* tab, or even directly in the chat box.

Every MMM script command must start with _!mmm_ – that's how Roll20 knows to send what follows to the MMM scripting engine. The _!mmm_ prefix must be followed by an MMM command (described below).

Scripts can be as short a single command:

| Line | Commands | What happens?
| ---- | -------- | -------------
| 1    | _!mmm_ **chat:** Hello World! | ***Finn:*** Hello World!

You can also put a whole sequence of commands together, one after another in separate lines:

| Line | Commands | What happens?
| ---- | -------- | -------------
| 1    | _!mmm_ **chat:** Oh dear, I'm pretty banged up. | ***Finn:*** Oh dear, I'm pretty banged up.
| 2    | _!mmm_ **do** setattr(sender, "LP", getattrmax(sender, "LP")) | *(set LP attribute to its maximum)*
| 3    | _!mmm_ **chat:** /me is back at ${getattr(sender, "LP")} points. | ***Finn is back at 25 points.***

Each of these commands would be executed as soon as the MMM scripting engine receives it. That's possible because each of the commands above is *self-contained:* It has everything it needs to execute in the same line.

However, some commands enclose a *block* of other commands – for example, if you want to *conditionally* execute some commands:

| Line | Commands | What happens?
| ---- | -------- | -------------
| 1    | _!mmm_ **if** getattr(sender, "LP") < 5 | *(check LP attribute)*
| 2    | _!mmm_     **chat:** /me is nearly dead! | ***Finn is nearly dead!***
| 3    | _!mmm_ **end if**

In the previous example, **if** is a block command: Every commands that follow the **if** command down to the **end if** command is a *block* of commands that's only going to be executed if the **if** condition is true. In general, every block command ends with a corresponding **end *block*** command.

You may have noticed that the **chat:** command in the example above has been moved to the right (by inserting extra spaces) in relation to the **if** and **end if** commands. That's completely up to you! Extra spaces between the _!mmm_ prefix and the command (and, in general, anywhere else) are ignored. *Indenting* the commands in a block relative to their enclosing block command is a time-honored practice among programmers – it simply makes the script's structure easier to see.

As you're writing longer and more complex scripts, it's good practice to enclose them in a **script** block:

| Line | Commands | What happens?
| ---- | -------- | -------------
| 1    | _!mmm_ **script**
| 2    | _!mmm_     **chat:** /me rolls [[1d20+12]] attack! | ***Finn rolls `29` attack!***
| 3    | _!mmm_     **if** $[[0]] >= 20 | *(check first roll from previous line)*
| 4    | _!mmm_         **chat:** Eat these [[1d6]] damage, evil foe! | ***Finn:*** Eat these `5` damage, evil foe!
| 5    | _!mmm_     **end if**
| 6    | _!mmm_ **end script**

Using a **script** block is optional, but if you do, it gives you the following benefits:

- The **script** block resets any unfinished previous block command you may have sent to the MMM scripting engine before (e.g. an **if** you sent earlier that you accidentally left without an **end if** to finish it up).
- Script execution is delayed until **end script** is received.
- If anything goes wrong inside the **script** block (e.g. you have a typo in a command), you'll get just one error message and the entire script won't be executed.

Inside a **script** block, you can use *variables*.

*Variables* allow you to store roll results or other computed values so you can refer back to them any future point in the **script** block:

| Line | Commands | What happens?
| ---- | -------- | -------------
| 1    | _!mmm_ **script**
| 2    | _!mmm_     **set** AttackRoll = [[1d20+12]] | *(assign roll result to variable)*
| 3    | _!mmm_     **if** AttackRoll >= 20 | *(check if attack roll was successful)*
| 4    | _!mmm_         **chat:** Attack with ${AttackRoll} dealing [[1d6]] damage | ***Finn:*** Attack with `23` dealing `3` damage
| 5    | _!mmm_     **else**
| 6    | _!mmm_         **chat:** Attack with ${AttackRoll} failed | *(not executed)*
| 7    | _!mmm_     **end if**
| 8    | _!mmm_ **end script**

You can freely choose your variable names as long as they only contain letters (A-Z), digits (0-9), and underscores (_). Variable names cannot contain other punctuation or spaces. Upper- and lowercase spelling is significant (so attackRoll, AttackRoll, and attackroll are three different variables).

Once set, the lifetime of variables only extends to the next **end script** command. If you want to save state between different scripts or script runs, consider using character attributes.


### _!mmm_ **script** [...] **end script**

Defines a script that's executed as a whole.

The **script** command also flushes any previous incomplete commands that might still be in the chat queue. This means that a script wrapped in a **script** block will reliably execute regardless of whatever happened before. It also means it's impossible to nest **script** blocks, but since the **script** command does nothing else, that's never necessary anyway.

See above (and below) for examples.


### _!mmm_ **chat:** *template*

Sends text to the chat impersonating the player or character who sent the script.

The template is simple text that can contain ${expression} placeholders. When the **chat** command is executed, all ${expression} placeholders are evaluated and substituted into the template text.

You can use /me, /whisper, and any other Roll20 chat directives in a **chat** command. 

| Line | Commands | What happens?
| ---- | -------- | -------------
| 1    | _!mmm_ **chat:** /me is bored. | ***Finn is bored.***
| 2    | _!mmm_ **chat:** Attacking with [[1d20+12]] | ***Finn:*** Attacking with `14`
| 3    | _!mmm_ **chat:** My half-life is ${getattr(sender, "LP") / 2}. | ***Finn:*** My half-life is 11.5.


### _!mmm_ **combine chat** [...] **end combine**

You can wrap a **combine chat** block around **chat** and other commands to combine all **chat** outputs within that block into a single chat output line. That's useful if you want to build a longer chat message with conditional parts – or if you'd just like to break a really long message line up into something that's easier to read.

| Line | Commands | What happens?
| ---- | -------- | -------------
| 1    | _!mmm_ **combine chat**
| 2    | _!mmm_     **chat:** Attack with [[1d20+12]] | *(queue message part containing attack roll)*
| 3    | _!mmm_     **if** $[[0]] >= 20 | *(check attack roll)*
| 4    | _!mmm_         **chat:** dealing [[1d6]] damage | *(queue message part containing damage roll)*
| 5    | _!mmm_     **else**
| 6    | _!mmm_         **chat:** failed | *(not executed)*
| 7    | _!mmm_     **end if**
| 8    | _!mmm_ **end combine** | ***Finn:*** Attack with `23` dealing `2` damage


### _!mmm_ **combine chat using** *expression* [...] **end combine**

By default, the parts of the message are separated with spaces when the **combine chat** command builds the final chat message. If you'd like something different to separate the parts (e.g. nothing, or a custom separating character or character sequence), use the **combine chat using** variant of the command:

| Line | Commands | What happens?
| ---- | -------- | -------------
| 1    | _!mmm_ **combine chat using** "" | *(evaluate separator expression: empty string)*
| 2    | _!mmm_     **chat:** Finn M | *(queue message part)*
| 2    | _!mmm_     **chat:** acRath | *(queue message part)*
| 2    | _!mmm_     **chat:** gar | *(queue message part)*
| 8    | _!mmm_ **end combine** | ***Finn:*** Finn MacRathgar

-----

### _!mmm_ **if** *expression* [...] **else if** *expression* [...] **else** [...] **end if**

Executes other commands conditionally.

The **if** and **else if** commands evaluate their expression (which will usually contains a comparison of some sort), and...

- If the result is true, execute the commands that follow until the next **else if**, **else**, or **end if** on the same level is reached, and then skip everything down to the **end if** command that completes the **if** block.
- If the result is false, skip the commands that follow until the next **else if**, **else**, or **end if** on the same level is reached.

You can use any commands inside an **if** block included other, nested **if** blocks.

That said – anything that's evaluated by the Roll20 macro engine *before* the command is sent to the MMM scripting engine will be always be executed unconditionally by Roll20 before MMM gets a shot at it. So while you absolutely *can* use inline rolls like `[[1d6]]`, roll queries like `?{Bonus|0}`, and attribute calls like `@{Finn|LP}` in your scripts, you *cannot* make them conditional by placing them inside an **if** block.

| Line | Commands | What happens?
| ---- | -------- | -------------
| 1    | _!mmm_ **set** Health = @{Finn\|LP} | *(assign current LP to Health variable)*
| 2    | _!mmm_ **if** Health >= 0.8 * @{Finn\|LP\|max} | *(check if 80% healthy or more)*
| 3    | _!mmm_     **chat:** I feel sufficiently envigorated!
| 4    | _!mmm_ **else if** ?{Heal if necessary?\|yes,true\|no,false} | *(less than 80% healthy – use result of roll query)*
| 5    | _!mmm_     **chat:** /me gulps a healing potion for [[1d6]] health. | *(chat with inline roll result)*
| 6    | _!mmm_     **set** Health = min(Health + $[[0]], @{Finn\|LP\|max}) | *(calculate new Health limited to max)*
| 7    | _!mmm_     **do** setattr("Finn", "LP", Health) | *(set LP to updated Health)*
| 8    | _!mmm_ **else**
| 3    | _!mmm_     **chat:** Feeling bad and no healing potion. Woe is me!
| 8    | _!mmm_ **end if**

In the example above, the "Heal if necessary?" question will pop up even before the entire script runs – actually, just when the **else if** line is received by the Roll20 chat engine and before it's sent to the MMM scripting engine. So this question will be asked even if Finn is sufficiently healthy to not even want healing. Unfortunately, the only way to make roll queries conditional is with conventional (and limited) Roll20 macro magic.


### _!mmm_ **set** *variable* = *expression*

Evaluates an expression and assigns its results to a variable. If the variable doesn't exist yet, it's created; if it already exists, its previous value is replaced.

| Line | Commands | What happens?
| ---- | -------- | -------------
| 1    | _!mmm_ **set** CurCount = getattr(sender, "AmmoCount") | *(stores AmmoCount attribute value in CurCount variable)*
| 2    | _!mmm_ **set** MaxCount = getattrmax(sender, "AmmoCount") | *(stores AmmoCount attribute max value in MaxCount variable)*
| 3    | _!mmm_ **set** FindProb = ?{Probability?\|100} / 100 | *(calculates FindProb variable from roll query result)*
| 4    | _!mmm_ **set** FindCount = (MaxCount - CurCount) * FindProb | *(calculates and assigns FindCount variable)*
| 5    | _!mmm_ **set** CurCount = CurCount + FindCount | *(updates CurCount variable)*


### _!mmm_ **do** *expression*

Evaluates an expression. This is useful if the expression has side effects, e.g. changes a character attribute. (If the expression returns a result, the result is discarded.)

| Line | Commands | What happens?
| ---- | -------- | -------------
| 1    | _!mmm_ **do** setattr(sender, "LP", min(getattr(sender, "LP") + [[1d6]], getattrmax(sender, "LP")) | *(add 1d6 LP, up to max LP)*
| 2    | _!mmm_ **do** setattr(sender, "AmmoCount", getattr(sender, "AmmoCount") - 1) | *(decrement AmmoCount attribute)*
| 3    | _!mmm_ **do** chat("/me likes chatting the hard way") | ***Finn likes chatting the hard way***



## Copyright & License

**Mych's Macro Magic** was created by Michael Buschbeck <michael@buschbeck.net> (2021).

It can be freely used and distributed under the [MIT License](https://choosealicense.com/licenses/mit/) provided this license and copyright notice are retained.
