+++
title = "Vim Keymap For Markdown Links"
date = 2026-02-05

[taxonomies]
tags = [ "markdown", "vim", "nvim", "neovim", "lua" ]

[extra]
toc = true
+++

I work with [Markdown](https://en.wikipedia.org/wiki/Markdown) a lot. I use it for note-taking, documenting projects, creating personal wikis *(and a world-building wiki!)*, etc.
And most recently, I have started this blog so I use it even more than before.

Whenever I edit text in Markdown, I often find myself adding links here and there.
Since I also use [Neovim](https://neovim.io/) as my primary text/code editor, I can easily add links to text elements using minimal vim motions.

It works well and it literally takes less than a second to do that.
But when I have to add dozens of links, it starts feeling a bit repetitive.

The great thing about Vim/Neovim: **You can configure your editor to do EXACTLY what you want**.
So, I thought: "why not create a custom keymap for that?"

What I wanted:

1. Copy some URL to my system clipboard
2. Select some text in Neovim
3. Type <kbd>&lt;leader&gt;l</kbd>
4. Have Neovim convert the selection to a markdown link `[text](link)`, where:
    - `text` is the selected text
    - `link` is the URL I copied to my system clipboard (`+` [register](https://vimhelp.org/change.txt.html#registers) in Vim)

# The keymap

Here's the keymap I have added to my Neovim config:

```lua,linenos
vim.keymap.set(
    "x",
    "<leader>l",
    "<Esc>`<i[<Esc>`>la](<C-r>+)<Esc>",
    { desc = "Apply markdown [L]ink on current selection" }
)
```

To define a new keymap, we call `vim.keymap.set(...)` and give it some arguments:

- The first argument `"x"` tells Neovim to only enable the keymap in [Visual mode](https://vimhelp.org/visual.txt.html#Visual-mode)
- The second argument `"<leader>l"` is the actual keybinding.
In this case, `<leader>` corresponds to the [Leader](https://vimhelp.org/map.txt.html#%3CLeader%3E) key which I set to <kbd>Space</kbd> in my config.
So, when I type <kbd>Space+l</kbd> it will trigger this keymap
- The third argument is the sequence of motions to execute when the keymap is invoked
- Finally, we define a description for the keymap

The thing we want to have a look at is the sequence of motions:

```
<Esc>`<i[<Esc>`>la](<C-r>+)<Esc>
```

So, let's break it down:

- First, we use `<Esc>` to switch to [Normal mode](https://vimhelp.org/intro.txt.html#Normal-mode)
- Then, we move the cursor to the start of the previous visual selection with `` `< ``
- After that, `i[<Esc>` lets us switch to [Insert mode](https://vimhelp.org/insert.txt.html#Insert-mode), put an opening square bracket '[', before going back to Normal mode once again with <kbd>Escape</kbd>
- Then, this time, we go to the end of the last visual selection with `` `>``
- We use `l` to move the cursor one unit to the right
  - *This is needed because the opening square bracket we inserted earlier has shifted the last visual selection. If we did not do that, we would be one character off*
- We proceed to append the ']' and '(' characters with `a](`
- While still in Insert mode, we use `<C-r>+` to paste the content of the `+` register, which is the system clipboard (where our URL is stored)
  - [`:help c_<C-R>`](https://vimhelp.org/cmdline.txt.html#c_%3CC-R%3E) to learn more
- Finally, we insert the closing parenthesis and go back to Normal mode with `)<Esc>`

That was not so bad, was it?

# Complementary keymap for Normal mode

The previous keymap works great for any visual selection.
It allows you to select some text and apply a markdown link to it.

But what if you only want to add a link to a single word?
Do we really want to bother with a visual selection for that?
I don't think so. Time to create a second keymap:

```lua,linenos
vim.keymap.set(
    "n",
    "<leader>l",
    '"9ciw[]<Esc>"9Pf]a(<C-r>+)<Esc>',
    { desc = "Apply markdown [L]ink on current word" }
)
```

It looks very similar to our first keymap with a few tweaks here and there.
The first difference we notice is the first argument `"n"` which tells Neovim to enable this keymap in Normal mode only.
The key binding has not changed, we still want to use `<leader>l`.

This means both keymaps use the same key binding, the only difference is the Vim mode you are in.
This is what determines which keymap to call.

Okay, let's go over the motions now:

```
"9ciw[]<Esc>"9Pf]a(<C-r>+)<Esc>
```

If you pay a closer look to the last part, you will notice that it is the same as before:

```
a](<C-r>+)<Esc>
```

So the only motions that differ are the ones prior to inserting the closing square bracket ']':

```
"9ciw[]<Esc>"9Pf]
```

New motions to break down. **Exciting!**

- Let's start with `"9ciw`:
  - `"9` tells Neovim we want to operate against the `9` register.
    There is no reason for using this specific register, you may use another one if you prefer.
    I picked this one arbitrarily.
    Just make sure not to pick register `+` because our URL is stored in there
  - Then, `ciw` will replace **inside** the current word and store the deleted text to register `9`
- The previous motion put us in Insert mode, we can now insert square brackets `[]` and go back to Normal mode with <kbd>Escape</kbd>
- The awaited return of register `9`! We use `"9P` to paste our deleted word to the **left** of the cursor
  - Had we use `p` instead of `P`, and the text would have been pasted outside of the square brackets *(which is not we want!)*
- Finally, we move the cursor on the next closing square bracket with `f]`
- Then, it's just the last part which is exactly the same as before: `a](<C-r>+)<Esc>`

So now, if we move the cursor on any word in Normal mode and hit `<leader>l`, it will automatically format it as a Markdown link. Pretty cool!

# Conclusion

Okay, this might seem like a lot just to save a few milliseconds here and there. Was it really worth it?

**YES!**
*(I have used `<leader>l` a lot to write this article already)*

Maybe it will take a lot of time before this keybind actually pays off.
But overall, I am having so much fun just trying to solve these kind of problems in Neovim.
This is what makes using Neovim so enjoyable to me.

I think having fun is the best remedy to avoid burnouts.
For me, I get my fun from using Neovim and the terminal, customizing my environment to be exactly how I want it.
It's probably different for you but I hope you get your fun from somewhere else at least.

I hope I made you learn some neat Vim tricks!
If you liked this article, please consider sharing it around to any other vim addicts.
If you want to take a peek at my Neovim config, I keep it on GitHub: [coko7/neovim-config](https://github.com/Coko7/neovim-config)

Feel free to check out my other articles too!
Until next time 👋

`:wqa`
