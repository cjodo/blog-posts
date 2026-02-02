## What is this plugin?

I found myself switching back and forth from my editor to my browser just to look up the conversion codes for colors or the conversions from px to rems in my css docs.  I thought it would be nice to have something just built into Neovim itself. After some looking around on github I couldn't find something that I really liked.  

So of course as developers do, I spent a good chunk of my summer after getting off work, just learning lua and browsing the neovim docs to write this plugin that saves a couple mouse clicks and keystrokes.

I was happy enough with the functionality to post it on the Neovim subreddit to see if anyone else would find it useful, surprisingly enough, it quickly got a few stars.

#### Current repo stars:
<img src="https://img.shields.io/github/stars/cjodo/convert.nvim?style=social" alt="GitHub stars">

I'm quite proud of it, and if you want to check it out.  Here's the repo: [https://github.com/cjodo/convert.nvim](https://github.com/cjodo/convert.nvim)

## What's next?
Since releasing it, I've added a few config options and added number systems as well to convert from binary <-> hex <-> octal.  I'm currently working on refactoring and adding a few more configuration options to help users make it their own.


## Docs:

## Dependencies
- [nui.nvim](https://github.com/MunifTanjim/nui.nvim): UI Components

## Features
- Convert css units with one simple command
- Base font supported for accurate rem conversion
- Convert all in a selection or entire buffer

## Installation: 
Use your favourite plugin manager

- Lazy: 
```lua
return {
    'cjodo/convert.nvim',
    dependencies = {
        'MunifTanjim/nui.nvim'
    },
    keys = {
        { "<leader>cn", "<cmd>ConvertFindNext<CR>", desc = "Find next convertable unit" },
        { "<leader>cc", "<cmd>ConvertFindCurrent<CR>", desc = "Find convertable unit in current line" },
        -- Add "v" to enable converting a selected region
        { "<leader>ca", "<cmd>ConvertAll<CR>", mode = {"n", "v"}, desc = "Convert all of a specified unit" },
    },
}
```
- Packer: 
```lua
use {
    'cjodo/convert.nvim',
    requires = { 'MunifTanjim/nui.nvim' },
    config = function()
        require('convert').setup()
        vim.keymap.set('n', '<leader>cn', '<cmd>ConvertFindNext<CR>', { desc = 'Find next convertible unit' })
        vim.keymap.set('n', '<leader>cc', '<cmd>ConvertFindCurrent<CR>', { desc = 'Find convertible unit in current line' })
        vim.keymap.set({ 'n', 'v' }, '<leader>ca', '<cmd>ConvertAll<CR>', { desc = 'Convert all of a specified unit' })
    end
}

```
## Commands:

| Command              | Description                                                               |
|----------------------|---------------------------------------------------------------------------|
| :ConvertFindNext     | Finds the next convertible unit                                           |
| :ConvertFindCurrent  | Finds the convertible unit in the current line (starting from cursor)     |
| :ConvertAll          | Converts all instances in a buffer or visual mode selection of one unit to another of the same type (size, color)                         |

## Usage
You can choose you're own custom keys for the ui menu

```lua
config = function()
    local convert = require('convert')
    -- defaults
    convert.setup({
        keymaps = {
            focus_next = { "j", "<Down>", "<Tab>" },
            focus_prev = { "k", "<Up>", "<S-Tab>" },
            close = { "<Esc>", "<C-c>", 'qq' },
            submit = { "<CR>", "<Space>" },
        },
        modes = { "color", "size", "numbers" } -- available conversion modes
    })
end

```
## Supported Conversions

### Size Units 📏  

| Unit | Description |
|------|------------|
| `px`  | Pixels |
| `rem` | Relative to root element |
| `cm`  | Centimeters |
| `in`  | Inches |
| `mm`  | Millimeters |
| `pt`  | Points |
| `pc`  | Picas |

---

### Color Formats 🎨  

| Format | Description |
|--------|------------|
| `rgb`  | Red-Green-Blue |
| `hex`  | Hexadecimal color code |
| `hsl`  | Hue-Saturation-Lightness |

---

### Number Systems 🔢  (Experimental)
- Defined with respective prefix: 0b, 0x, 0o

| Format       | Description |
|-------------|------------|
| `bin`       | Binary |
| `hexadecimal` | Hexadecimal |
| `octal`     | Octal |
