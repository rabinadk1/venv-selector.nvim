<p align="center">
  <h1 align="center">:tada: Python Venv Selector</h2>
</p>

<p align="center">
	A simple neovim plugin to let you choose what virtual environment to activate in neovim.
</p>

<p align="center">
    <img src="venv-selector.png" />
</p>

# ⚡️ Features

- Switch back and forth between virtual environments without restarting neovim
- New and much more flexible configuration to support finding the exact venvs you want.
- Browse existing python virtual environments on your computer and select one to activate inside neovim.
- Supports **all** virtual environments using configurable **regular expressions expressions**, such as:
  - [Python](https://www.python.org/) (`python3 -m venv venv`)
  - [Poetry](https://python-poetry.org)
  - [PDM](https://github.com/pdm-project/pdm)
  - [Pipenv](https://pipenv.pypa.io/en/latest/)
  - [Anaconda](https://www.anaconda.com)
  - [Pyenv](https://github.com/pyenv/pyenv)
  - [Virtualenvwrapper](https://virtualenvwrapper.readthedocs.io/en/latest/)
  - [Hatch](https://hatch.pypa.io/latest/)
- Supports callbacks to further filter or rename telescope results as they are found.
- Supports using any program to find virtual environments (`fd`, `find`, `ls`, `dir` etc)
- Supports running any interactive command to populate the telescope viewer:
  - `:VenvSelect fd 'python$' . --full-path -IH -a`

- Support [Pyright](https://github.com/microsoft/pyright), [Pylance](https://github.com/microsoft/pylance-release) and [Pylsp](https://github.com/python-lsp/python-lsp-server) lsp servers with ability to config hooks for others.
- Cached virtual environment that ties to your current working directory for quick activation
- Requires [fd](https://github.com/sharkdp/fd) and [Telescope](https://github.com/nvim-telescope/telescope.nvim) for fast searches, and visual pickers.
- Requires [nvim-dap-python](https://github.com/mfussenegger/nvim-dap-python), [debugpy](https://github.com/microsoft/debugpy) and [nvim-dap](https://github.com/mfussenegger/nvim-dap) for debugger support


#### **NOTE:** This regexp branch of the plugin is a rewrite that works differently under the hood to support more advanced features. Its under development and not ready for public use yet.

## Configuration snippet for [lazy.nvim](https://github.com/folke/lazy.nvim)

```
{
  "linux-cultist/venv-selector.nvim",
    dependencies = {
      "neovim/nvim-lspconfig", 
      "mfussenegger/nvim-dap", "mfussenegger/nvim-dap-python", --optional
      { "nvim-telescope/telescope.nvim", tag = "0.1.6", dependencies = { "nvim-lua/plenary.nvim" } },
    },
  lazy = false,
  branch = "regexp", -- This is the regexp branch, use this until its merged with the main branch later
  config = function()
      require("venv-selector").setup()
    end,
    keys = {
      { ",v", "<cmd>VenvSelect<cr>" },
    },
},
```

## Why did you rewrite the plugin?

Because the current code has grown from supporting only simple venvs to lots of different venv managers. Each one works in a slightly different way, and the current code has lots of conditional logic to try and figure out what to do in certain situations. It made it difficult to change something without breaking something else. And it made it difficult to add features in a clean way. 

This rewrite is about giving you as a user the power to add your own searches, and have anything you want show up in the telescope viewer. If its the path to a python executable, the plugin will attempt to activate it. Note that your LSP server must be running for this to happen, so you need to have a python file opened in the editor. 

## Default searches

A default search is one that the plugin does automatically.

These are designed to find venvs in your current working directory and from different venv managers in their default paths.

Some of them use the special variables `$CWD` and `$WORKSPACE_PATH`. You can also use these in your own searches. They will contain the value of your neovim current working directory and the neovim workspace directories when the LSP is active.

### The current default searches are for:

- Venvs created by [Virtualenvwrapper](https://virtualenvwrapper.readthedocs.io/en/latest)
- Venvs created by [Poetry](https://python-poetry.org)
- Venvs created by [Hatch](https://hatch.pypa.io/latest)
- Venvs created by [Pyenv](https://github.com/pyenv/pyenv)
- Venvs created by [Anaconda](https://www.anaconda.com)
- Venvs in the current working directory
- Venvs in the lsp workspace directories

The search patterns are defined here: https://github.com/linux-cultist/venv-selector.nvim/blob/regexp/lua/venv-selector/config.lua

If your venvs are not being found because they are in a custom location, you can easily add your own searches to your configuration.

## My venvs dont show up - how can i create my own search?

You create a search for python venvs with `fd` and you put that into the plugin config. You can also use `find` or any other command as long as its output lists your venvs. 

The best way to craft a search is to run `fd` with your desired parameters on the command line before you put it into the plugin config.

The configuration looks like this:

```
      require("venv-selector").setup {
        settings = {
          search = {
            my_venvs = {
              command = "fd python$ ~/Code",
            },
          },
        },
      }

```
The example command above launches a search for any path ending with `python` in the `~/Code` folder. Its using a regular expression where `python$` means the path must end with the word python. For windows we would need to use `python.exe$` instead. Here are the results:

```
/home/cado/Code/Personal/databricks-cli/venv/bin/python
/home/cado/Code/Personal/dbt/venv/bin/python
/home/cado/Code/Personal/fastapi_learning/venv/bin/python
/home/cado/Code/Personal/helix/venv/bin/python
```


These results will be shown in the telescope viewer and if they are a python virtual environment, they can be activated by pressing enter. 

You can add multiple searches as well:

```
      require("venv-selector").setup {
        settings = {
          search = {
            find_code_venvs = {
              command = "fd /bin/python$ ~/Code --full-path",
            },
            find_programming_venvs = {
              command = "fd /bin/python$ ~/Programming/Python --full-path -IHL -E /proc",
            },
          },
        },
      }
```

## Common flags to fd


| Fd option             | Description |
|-----------------------|-------------|
| `-I` or `--no-ignore` | Ignore files and directories specified in `.gitignore`, `.fdignore`, and other ignore files. This option forces `fd` to include files it would normally ignore. |
| `-L` or `--follow`    | Follow symbolic links while searching. This option makes `fd` consider the targets of symbolic links as potential search results. |
| `-H` or `--hidden`    | Include hidden directories and files in the search results. Hidden files are those starting with a dot (`.`) on Unix-like systems. |
| `-E` or `--exclude`   | Exclude files and directories that match the specified pattern. This can be used multiple times to exclude various patterns. |

So if you dont add `-I`, paths that are in a `.gitignore` file will be ignored. Its common to have venv folders in that file, so thats why this flag can be important.

However, some flags slows down the search significantly and should not be used if not needed (like `-H` to look for hidden files). If your venvs are not starting with a dot in their name, you dont need to use this flag.




### Override or disable a default search

If you want to **override** one of the default searches, create a search with the same name. This changes the default workspace search. 

`settings = {
  search = {
    workspace = {
      command = "fd /bin/python$ $WORKSPACE_PATH --full-path --color never -E /proc -unrestricted",
    }
  }
}`

The above search adds the unrestriced flag to fd. See `fd` docs for what it does!

If you want to **disable one** of the default searches, you can simply set it to false. This disables the workspace search.

`settings = {
  search = {
    workspace = false
  }
}`

If you want to **disable all** built in searches, set the global option `enable_default_searches` to false (see separate section about global options)


## Changing the output in the telescope viewer

Maybe you dont want to see the entire full path to python in the telescope viewer. You can change whats being displayed by using a callback function.

```
{
  "linux-cultist/venv-selector.nvim",
    dependencies = {
      "neovim/nvim-lspconfig", 
      "mfussenegger/nvim-dap", "mfussenegger/nvim-dap-python", --both are optionals for debugging
      { "nvim-telescope/telescope.nvim", tag = "0.1.6", dependencies = { "nvim-lua/plenary.nvim" } },
    },
  lazy = false,
  branch = "regexp", -- This is the regexp branch, use this until its merged with the main branch later
  config = function()
  
      -- This function gets called by the plugin when a new result from fd is received
      -- You can change the filename displayed here to what you like. Here in the example we remove the /bin/python part.
      local function remove_last_part(filename)
        return filename:gsub("/bin/python", "")
      end


      require("venv-selector").setup {
        settings = {
          options = {
            -- If you put the callback here as a global option, its used for all searches (including the default ones by the plugin)
            on_telescope_result_callback = remove_last_part 
          },

          search = {
            my_venvs = {
              command = "fd python$ ~/Code", -- Sample command, need to be changed for your own venvs
              
              -- If you put the callback here, its only called for your "my_venvs" search
              on_telescope_result_callback = remove_last_part 
            },
          },
        },
      }
    end,
    keys = {
      { ",v", "<cmd>VenvSelect<cr>" },
    },
},
```
## Python debugger support with 




## Global options to the plugin

```
search = {
  settings = {
    options = {
      debug = false                       -- switches on/off debug output
      on_telescope_result_callback = nil  -- callback function for all searches
      fd_binary_name = fd                 -- plugin looks for `fd` or `fdfind` but you can set something else here
      enable_default_searches = true      -- switches all default searches on/off
    }
  }
}
```

More docs coming up soon!

