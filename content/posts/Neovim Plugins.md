+++
date = '2025-02-08T23:28:37+05:00'
title = 'Neovim Complete Setup and Integrating ChatGPT'
tag = ["Neovim","Lua"]
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

2. For C/C++ development, ensure clangd is installed and configured:
Add the following piece of code to init.lua after the ending of above mentioned code:
```lua
--clangd server setup
local lspconfig = require('lspconfig')
lspconfig.clangd.setup({
  cmd = { "clangd" },  -- Ensure this points to the system-installed clangd
  -- Additional configuration options can be added here
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
 4. Open nvim and install lsp for any language, I am doing for C/C++:
  ```python
   :MasonInstall clangd
```
> In MasonInstall menu, Press U to update and arrow key to scroll and find your required lsp

 5. Update packages using:
 ```lua
  :Lazy update    
```
 ```lua
  :Lazy sync
```

