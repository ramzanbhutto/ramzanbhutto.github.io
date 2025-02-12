+++
date = '2025-02-08T23:28:37+05:00'
title = 'Neovim Complete Setup and Integrating ChatGPT'
tag = ["Neovim","Lua","ChatGPT"]
+++
---

## Part 1: Setting Up NVChad

### Step 1: Removing any previous configuration

1. Linux/MacOS Users
```python
rm -rf ~/.config/nvim
rm -rf ~/.local/state/nvim
rm -rf ~/.local/share/nvim
```
2. Flatpak(Linux) Users
```python
rm -rf ~/.var/app/io.neovim.nvim/config/nvim
rm -rf ~/.var/app/io.neovim.nvim/data/nvim
rm -rf ~/.var/app/io.neovim.nvim/.local/state/nvim
```
3. Window Users
 - CMD
    ```python
     rd -r ~\AppData\Local\nvim
     rd -r ~\AppData\Local\nvim-data
   ```
 - Powershell
   ```python
   rm -Force ~\AppData\Local\nvim
   rm -Force ~\AppData\Local\nvim-data
   ```
### Step 2: Installing NVChad
1. Clone the NVChad repository
 - Linux/MacOS:
 ```python
   git clone https://github.com/NvChad/starter ~/.config/nvim && nvim
 ```
 - Flatpak:
 ```python
  git clone https://github.com/NvChad/starter ~/.var/app/io.neovim.nvim/config/nvim && flatpak run io.neovim.nvim
 ```
 - Windows (CMD):
 ```python
   git clone https://github.com/NvChad/starter %USERPROFILE%\AppData\Local\nvim && nvim
 ```
 - Windows (Powershell):
 ```python
   git clone https://github.com/NvChad/starter $ENV:USERPROFILE\AppData\Local\nvim && nvim
```
### Step 3: Basic Neovim commands to attempt further processes
 Type `nvim` or `neovim` on the terminal before any file name(optional) to enter into neovim environment.
1. Enter editing mode:
```python
  i
```
2. Escape the editing mode to save or exit:
> Press the escape button
3. Save file:
 ```python
 :w
```
4. Exit with saving:
```python
  :wq
```
5. Exit without saving:
```python
  :q!
```

### Step 4: Essential processes after installing
 1. Run following command into nvim after lazy.nvim finishes downloading plugins:
 ```python
  :MasonInstallAll
 ```
 2. Navigate to nvim directory in `.config` folder to the remove `.git` folder.
 ```python
   rm -rf /home/username/.config/nvim/.git
```
 3. Learn customization of UI and base46 by using following command inside nvim:
 ```python
 :h nvui
```
 4. Update the packages:
 ```python
  :Lazy sync
```

### Step 5: Setting up LSP Configurations
 You have to add required language server protocols to improve your coding experience.
 1. Ensure that `lazy_config` is set up correctly after `vim.otp.rtp` in the file `home/username/.config/nvim/init.lua`, if not, then add the following content to `init.lua` file after `vim.otp.rtp`:
 ```lua
local lazy_config = require "configs.lazy"
-- load plugins
require("lazy").setup({
  {
    "NvChad/NvChad",
    lazy = false,
    branch = "v2.5",
    import = "nvchad.plugins",
  },
  { import = "plugins"},
}, lazy_config)
```

2. For C/C++ and Python development, ensure clangd and pyright is installed and configured:
Add the following piece of code to init.lua after the ending of above mentioned code:
```lua
--clangd server setup
local lspconfig = require('lspconfig')
lspconfig.clangd.setup({           --For C/C++
  cmd = { "clangd" },  -- Ensure this points to the system-installed clangd
  -- Additional configuration options can be added here
})
lspconfig.pyright.setup({      --For Python
  autoSearchPaths = true,
  diagnosticMode = "workplace",   --you can also set it to "openFilesOnly" to check errors in only oepned files
})

```
3. `Edit lazy.lua` file to enable plugins and comment the lines after disabled_plugins lines like mentioned below:
 > Address: /home/username/.config/nvim/lua/configs/lazy.lua

```lua
  performance = {
    rtp = {
      disabled_plugins = {
        --"2html_plugin",
        --"tohtml",
        --"getscript",
        --"getscriptPlugin",
        --"gzip",
        --"logipat",
        --"netrw",
        --"netrwPlugin",
        --"netrwSettings",
        --"netrwFileHandlers",
        --"matchit",
        --"tar",
        --"tarPlugin",
        --"rrhelper",
        --"spellfile_plugin",
        --"vimball",
        --"vimballPlugin",
        --"zip",
        --"zipPlugin",
        --"tutor",
        --"rplugin",
        --"syntax",
        --"synmenu",
        --"optwin",
        --"compiler",
        --"bugreport",
        --"ftplugin",
     },
    },
  },

```
 4. Open nvim and install lsp for any language, I am doing for C/C++ and Python:
  ```python
   :MasonInstall clangd
   :MasonIntall python-lsp-server
   :MasonInstall pyright
```
> In MasonInstall menu, Press U to update and arrow key to scroll and find your required lsp

 5. Update packages using:
 ```lua
  :Lazy update    
```
 ```lua
  :Lazy sync
```

---

## Part 2: Integrating ChatGPT into Neovim

### Step 1: Get the API Key

 1. Click [here](https://platform.openai.com/docs/overview) and Sign up or Login in for OPENAI Platform.
  
 2. Click on the setting ⚙️  option and Navigate to API keys.

 3. Click on `+ Create new secret key` and fill some sections to generate your key. 

 4. After generating, copy your key.

### Step 2: Save into your zsh or bash file

 1. Verify your terminal type by typing following command into your terminal:
```lua
echo $0
```
>if it indicates `/bin/zsh` or `zsh` then you're a zsh terminal user, if it indicates `/bin/bash` or `bash` then you're bash terminal user.

 2. If you're a zsh terminal user, open `.zshrc` file or if you're a bash terminal user, open `.bashrc` file into your home directory and add the following lines:
 ```lua
  export OPENAI_API_KEY="paste_your_api_key_here"
```

 3. Source the file after making changes and pasting your OPENAI_API_KEY key, by running following command into terminal:
 ```lua
  source .zshrc
```
> or
```lua
source .bashrc
```
 4. Verify your API key:
 ```lua
 echo $OPENAI_API_KEY
```

### Step 3: Configuring the init.lua file
 >Address: /home/username/.config/nvim/init.lua

 Locate the following piece of code into init.lua file:
```lua
   {
   "NvChad/NvChad",
   lazy = false,
   branch = "v2.5",
   import = "nvchad.plugins",
   },
```
Then, add the following beautiful text just after it:
```lua
     {
    "jackMort/ChatGPT.nvim",
    event = "VeryLazy",
    dependencies = {
      "MunifTanjim/nui.nvim",
      "nvim-lua/plenary.nvim",
      "nvim-telescope/telescope.nvim",
      "folke/trouble.nvim",
    },
      config = function()
      require("chatgpt").setup({
        -- Your ChatGPT.nvim configuration
        api_key_cmd = "echo $OPENAI_API_KEY",

        vim.api.nvim_set_keymap("n", "<C-s>", "<cmd>lua require('chatgpt').submit()<CR>", { noremap = true, silent = true }),-- Submit text in the ChatGPT prompt
        vim.api.nvim_set_keymap("n", "<C-c>", "<cmd>lua require('chatgpt').close()<CR>", { noremap = true, silent = true }), -- Close ChatGPT window
        vim.api.nvim_set_keymap("n", "<C-y>", "<cmd>lua require('chatgpt').yank_last()<CR>", { noremap = true, silent = true }), -- Yank last ChatGPT response
        vim.api.nvim_set_keymap("n", "<C-u>", "<cmd>lua require('chatgpt').scroll_up()<CR>", { noremap = true, silent = true }), -- Scroll up
        vim.api.nvim_set_keymap("n", "<C-d>", "<cmd>lua require('chatgpt').scroll_down()<CR>", { noremap = true, silent = true }), -- Scroll down
        vim.api.nvim_set_keymap("n", "<C-e>", "<cmd>lua vim.api.nvim_screenshot('/home/ramzan/screenshot')<CR>", { noremap = true, silent = true}),
        vim.api.nvim_set_keymap("n", "<C-2>", "<cmd>lua require('chatgpt').goto_next_end()<CR>", { noremap = true, silent = true}),
        vim.api.nvim_set_keymap("n", "<C-w>", "<cmd>lua require('chatgpt').close()<CR>", { noremap = true, silent = true})
      })
    end,
  },
```

> This will automatically set the keybindings which are essential to run ChatGPT into it. Save this file and exit.

### Step 4: Final setup

1. Open Neovim and update the lazy package:
```lua
 :Lazy update
```
```lua
 :Lazy sync
```
2. Shortcuts to run ChatGPT:
 - Open Neovim
 - Open ChatGPT menu:
 ```lua
 :ChatGPT
```
 - It will open a menu inside our terminal, then give any command to it.
 - Press `Esc` and then press `enter` to submit the command.
 - Press `Esc` and then press `ctrl+C` or enter `:q` to exit ChatGPT.

 3. You can further learn neovim Shortcuts by:
 ```lua
  :h nvui
```

---

>That's it. Finally the setup is over and chatgpt is working inside our terminal. If any issue persists like `Incorrect OPENAI_API_KEY` then you just have to source the bash or zsh file `source .zshrc` or `source.bashrc`. Happy coding 💟
