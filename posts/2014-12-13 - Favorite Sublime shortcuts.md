Sublime Text 2 offers three wonderful ways to increase your productivity: shortcuts, snippets, and macros. They are absolutely worth taking the time to master and explore.

# Shortcuts

Here is a collection of my favorite Sublime shortcuts. They massively improve my speed. I'll focus on some of the less obvious ones that I haven't seen commonly used among my cohort.

**Selecting things**

* `⌘ + Shift + Space` - select the enclosing scope (e.g. everything contains in {}), can press multiple times to highlight containing brackets and other scopes
* `⌘ + "mouse_click"` - set multiple cursors for editing
* `⌘ + D` - select instances of the current word and set cursors on them
* `⌘ + L (with lines highlighted)` - place cursors at the end of selected lines
* `^ + Shift + Up/Down` - place cursor on line above or below your cursor

**Navigating to things**

* `⌘ + R` - go to 'symbols' such as function names
* `^ + G` - go to specific line
* `^ + L` - visually center current line

**Editing things**

* `^ + K` - delete the current line
* `⌘ + Shift + V` - paste with proper indentation
* `⌘ + C (nothing highlighted)` - copies *the whole line* -- what?!
* `⌘ + X (nothing highlighted)` - cuts the whole line
* `⌘ + Shift + D` - duplicates line

These are a handful less commonly used but amazingly useful; if you can master them, you'll look like a wizard when you code (unless your coding is charisma-based, in which case you'll look like a sorcerer).

Looking for more? Browse through the toolbars. 

# Customize your world

Sublime Text provides access to "macros" and "snippets" to improve productivity, both of which are highly customizable. They can sometimes provide the same functionality, but here's essentially what they're intended for:

    **Snippets**: template code, conjured by typing in a key sequence and pressing `Tab`. For example: typing `for` and then `Tab` produces a standard `for` loop.
    **Macros**: a shortcut that performs a sequence of key commands. For example: go to end of line, insert semicolon, return to where you were before.

Tinkering with macros and snippets can seem like a daunting task. I promise you will be glad you did. You can learn in small blocks -- don't expect to customize everything all at once. If you're concerned about it being a waste of time, I point you to this XKCD comic.

{<13>}![xkcd comic](http://imgs.xkcd.com/comics/is_it_worth_the_time.png)

(Source: [XKCD](http://xkcd.com/1205/))

# Snippets

Snippets are the heart of keystroke savings for any programmer. Yes, we can all write out `function(){}` perfectly well; now let's produce that by typing `f` + `Tab` and get the same thing. Way better.

I have only begun to brush the surface of snippets and their power. There are many packages out there such as "JavaScript Snippets" that give you a huge collection to work with. But, before exploring those, I suggest you play around with the standard set.

The neat thing about snippets is that you can tab around to standard locations for editing. Create an `if` snippet, you'll be directed to the test, then hit tab and you'll move to the code block.

    if ( // you start here ) { // and tab to end here }

Sometimes there are multiple such fields. Keep track of the display in the lower left corner to see where you are. To exit out of the field navigation mode, use an arrow key.

{<14>}![fields display](/content/images/2014/12/Screen-Shot-2014-12-14-at-12-13-29-PM.png)

Rather annoyingly, many of the standard JS snippets feature semicolons where they don't belong. Take the `for` snippet for instance.

    if (true) {};

That little semicolon on the end make code style enthusiasts cry.

Tess Myers (also of Hack Reactor) wrote a [tidy blog post](http://techphrasis.azurewebsites.net/want-to-fix-your-sublime-for-loop/) about how to get rid of them. I suggest you follow her directions and purge them! She also writes about how to create your own.

Here's a bunch I use constantly:

* `for` - for loop
* `if` - if test
* `ife` - if/else test
* `f` - anonymous function
* `:f` - method function
* `proto` - prototype method

Hungry for more? Explore your current snippets available:

{<15>}![snippets toolbar](/content/images/2014/12/snippets.png)

Or install more using your Package Manager.

# Macros

There are two ways to make macros: recording, or writing manually. To record, go to "Tools -> Record Macro." Enter the sequence of key commands to perform. Then go to "Tools -> Stop Recording Macro." You can replay it to ensure it does what you want, then save it. 

You'll then need to assign a keybinding to run the macro file. Go to "Sublime Text 2 -> Preferences -> Key Bindings - User". 

{<16>}![key bindings menu](/content/images/2014/12/key-bindings.png)

Like most things in ST2, the key bindings are JSONs. Here's the general format to set up a macro:

    {
      "keys": ["<keys you want>"], "command": "run_macro_file", "args": {"file": "Packages/User/<name_of_file>.sublime-macro"}
    },
    { 
      rinse and repeat
    }

You'll replace `<keys you want>` using this [list of possible keys](http://sublime-text-unofficial-documentation.readthedocs.org/en/latest/reference/key_bindings.html#bindable-keys), and `<name_of_file>` with the name of the saved macro. Multiple keys are separated by `+`. Visit [this document](http://sublime-text-unofficial-documentation.readthedocs.org/en/latest/reference/key_bindings.html#structure-of-a-key-binding) for more information.

# Macros - My Greatest Hits

Here are some of my favorites that I've built or learned from the Internet. You can either copy the code and save as `.sublime-macro` files in your `~/Library/Application Support/Sublime Text 2/Packages/User` folder, or reproduce by recording your own macro.

**Insert semi-colon at end of line**
Great for tagging on the semi-colon as you're working.
The code:

    [
        { "command": "set_mark" },
        { "command": "move_to", "args": {"to": "eol"} },
        { "command": "insert_snippet", "args": {"contents": "${TM_LINE_TERMINATOR:;}"} },
        { "command": "swap_with_mark" },
        { "command": "clear_bookmarks", "args": {"name": "mark"} }
    ]

The keybinding:

    {
      "keys": ["super+;"], "command": "run_macro_file", "args": {"file": "Packages/User/EndOfLineSemicolon.sublime-macro"}
    }

**Delete and finish with a tab**
Tired of deleting a tab *and then* clicking the tab button? Do them both at once, and get yourself on the correct indentation automatically!

    [
        {
            "args":
            {
                "file": "Packages/Default/Delete to Hard BOL.sublime-macro"
            },
            "command": "run_macro_file"
        },
        {
            "args": null,
            "command": "reindent"
        }
    ]

The keybinding:

      {
        "keys": ["super+backspace"], "command": "run_macro_file", "args": {"file": "Packages/User/DeleteLineFinishWithTab.sublime-macro"}
      }


** Clear the current line**
Editing in the middle of a line and want to throw it all away? Instead of going to the end, highlighting everything, and deleting, just use this macro.

The code:

    [
        {
            "args":
            {
                "to": "eol"
            },
            "command": "move_to"
        },
        {
            "args":
            {
                "extend": true,
                "to": "bol"
            },
            "command": "move_to"
        },
        {
            "args": null,
            "command": "left_delete"
        }
    ]


The keybinding:

    {
      "keys": ["super+shift+backspace"], "command": "run_macro_file", "args": {"file": "Packages/User/ClearLine.sublime-macro"}
    }

# Conclusion

Shortcuts, snippets, and macros all take time to learn, master, and customize to your tastes.. They are absolutely worth that time, however. Developing proficiency with these tools will allow you to blaze through repititious actions and code so you can focus on what matters: reasoning about the problem at hand.

# References

* [Official Sublime shortcuts](http://docs.sublimetext.info/en/latest/reference/keyboard_shortcuts_osx.html)






