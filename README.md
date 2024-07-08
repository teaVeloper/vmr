# vmr - **v**irtual environment **m**anage**r**

**WORK IN PROGRESS**
This is currently a DRAFT only, until the API design and its implementation is finished, this is a teaser of what i
want, and nothing else!
Once I am there, this Note will vanish.
-----

Python virtual environment manager, that is customizable to your preferences and needs and works with different
backends.
This is my opinionated version of how I like to manage my 'virtualenvs'.

It is meant as a wrapper on the tools you choose to use, similar to
[virtualenvwrapper](https://github.com/python-virtualenvwrapper/virtualenvwrapper) but with different API and design.
Still it is heavily inspired by its Ideas and its worth checking out!

## Installation

The easiest and recommended way is using 'pipx':
```sh
pipx install vmr
```

But you could use any other way, to get the tool in your system and available on `$PATH`

Now once its done, you need to add the 'shell-wrapers' in your 'shell-config' and optionally add the interactive-addons.


This is done via 
`vmr shellenv > shell-config-file`
or
`vmr shellenv --interactive > shell-config-file`

alternatively you can copy the scripts wherever you want, and if you do not supply a folder, it uses 'vmrs`
config-folder. These you can manually source, then the path is independent of the install path of 'vmr'
`vmr shellenv --copy [path]`


To verify the installation and config worked you can run
`vmr healthcheck`

### Dependencies

You need to have a working install of 'python'

Additionally at least one of the supported backends:
- [uv](https://github.com/astral-sh/uv)
- [virtualenv](https://github.com/pypa/virtualenv)
- [venv](https://docs.python.org/3/library/venv.html) - most likely already shipped with python, some OS decide to no
ship on system-python.


If you wish for interactivity on your shell, you need an 'interactive fuzzy-finder' such as:
- [fzf](https://github.com/junegunn/fzf)
- [fzy](https://github.com/jhawthorn/fzy)
- [skim](https://github.com/lotabout/skim)

## Design Ideas

I want to manage my 'virtual environments' independently of my projects and tools. I want them to be stored in a central
place - except I opt out.
I want to choose which toolchain I use and I want to have a convenient API that works seamlessly across my projects, no
matter what toolchain these projects use.

I really like the ideas of [virtualenvwrapper](https://github.com/python-virtualenvwrapper/virtualenvwrapper) but I dont
like the API of having different commands for different activities, and also it is missing some features I want to have
built in.
I like some ideas of how [poetry](https://python-poetry.org/) does handle management of virtualenvironments, but they
are too strongly tied to a project and lack flexibility, as well I do not support some other of poetry's design ideas.

I want the wrapper to be 'sourced' in the shell, as any workaround might break a users shell environment.

I want one easy to remember command `vmr` with expressive subcommands and options, a central config file and sensible
defaults (Its arguable what is sensible, its my preference really :P ).

This tool is designed for me to aid my workflow and preferences, if it finds adaption and support I am happy, if not I
am happy too.


--- 

`vmr` should utilize these different backends, for creating a 'virtualenv' the order is how it will try to search for
them, if not some is configured explicitely:
- uv
- virtualenv
- venv

locally it can (if not configured else) read your `pyproject.toml` and support:
- poetry
- hatch
- rye
- conda?

For python-versions it should support (depending on configuration and auto-explore)
- mise
- asdf
- pyenv
- conda
- system-python
It will search in 'default-paths' of these tools and `$PATH` for different python versions and utilize, but if anything
is configured explicitely, than of course that.


## Alternatives

You should give all these a look, maybe they serve your needs better than my project. After all I got inspiration from
all of them.

- [virtualenvwrapper](https://github.com/python-virtualenvwrapper/virtualenvwrapper) - covers most of my desired
features
- [poetry](https://python-poetry.org/) - handles some features on per project base
- [rye](https://github.com/astral-sh/rye) - handles some features on per project base?
- [hatch](https://github.com/pypa/hatch) - handles some features on per project base
- [miniconda](https://docs.anaconda.com/miniconda/) - just conda package manager
- [zpy](https://zpy.readthedocs.io/en/latest/) - zsh plugin with different ideas as mine, but different approach and
goals.

## Usage

Here are some examples of the main usage, configuration will be 'assumed' with default values, details in the next
sections.
Many of the behaviour can be adapted and customized with the correct options or CLI Flags, see [config](#Config) and
[referece](#Reference)

### Create a venv

```
assumptions:
folder_name: my_project
python_version: 3.9
```
Creation will also associate a project folder (with hash of the absolute path) with a venv.

**Plain**
`vmr create`
will create a 'venv' with name 'my_project-py3.9'


**Names**
`vmr create my-venv`
will create a 'venv' with name 'my-venv-py3.9'

### Activate
```
assumptions:
folder_name: my_project
python_version: 3.9
venv from before created
```
Activation does not need the 'full-qualifying' name, if there is no ambiguity

**Plain**
`vmr activate`
will activate 'associated' venv thus 'my_project-py3.9'

if multiple are associated, it will prompt an error and list all associated venvs. 


**Named**
`vmr activate my-venv`
will activate a 'venv' with name 'my-venv-py3.9'

lets assume, we created 'my-venv' with different python versions
```
my-venv-3.8
my-venv-3.9
```
then above will prompt an error and list the two available ones, asking to supply the 'full-qualifying' name

### Deactive

`vmr deactivate` or `vmr d` - it basically just sends `deactivate` in your shell.

### Delete a venv

`vmr rm` - deletes associated venv

`vmr rm my-venv` - deleted my-venv if not ambiguous

`vmr rm my-venv*` - deletes all venvs that are named 'my-venv-pyVersion'

### Create a tempenv

`vmr tmpenv` - creates a venv in `/tmp` (or else if configured differently) and activates it, for direct usage.

### list venvs

`vmr list` - will list all available venvs. There are filters and globbing patterns possible

### Garbage Collection

`vmr gc` - clean and delete orphaned (from 'tempenv') and old (the meaning of old is configured, by default None are old) environments.


## Configuration

'vmr' uses a config file in `toml` format, many of the 'core' options can be overwritten or alternatively set via
'environment variables'. 
On command call most of these options can also be set explicitely via 'cli flags'

precedence thus is:
'CLI Tags' > 'Environment Variables' > Config File.

Config from local `pyproject.toml` files will only be used (and overwrites global config file and environment variables)
if the option for local settings is set.

**Config location:** 'vmr' will search first if exists in `$VMR_CONFIG_HOME` if this varible is not set it will try
`$XDG_CONFIG_HOME/vmr` and if this is not set then `~/.config/vmr`.

The config-file can be named either `vmr.toml` or `config.toml`, but if both exist then `vmr.toml` will be used.

Here I will summarize the options that are defined per default and their settings, and the organization of the
config-file. No CLI-Flags will be mentioned here.

The notion 'backend' is a programm used to create a virtualenv. With 'driver' a supplier of python versions is meant.
Any options for the supported backends and drivers are given through to the subprocesses.

Here is a basic outline of the toml fields.
```toml
[global]
# Global configs
[activate]
# Configs per command
[create]
[rm]
[ui]
# Config UI
[backend]
# configuring backend
[backend.uv]
# Configs per backend
[backend.virtualenv]
[driver]
# configuring driver
[driver.asdf]
# configs per driver
[driver.system]
```

**VENV_PATH**
`global.venv_path` - `VMR_VENV_PATH` - `$XDG_DATA_HOME/vmr/venvs` if not set `~/.local/share/vmr/venvs`

**DEFAULT PACKAGES**
there are 2 Options, if both are set, both will be used - thus the packages in the list and the ones in the
'requirements file' will be installed.
`global.default_packges` a list
`global.default_requirements` - the path to a `requirements.txt` - `VMR_DEFAULT_REQUIREMENTS`

**EXAMPLE**
```toml
[global]
venv_path = "$XDG_DATA_HOME/vmr/virtualenvs"
default_packages = ["ipython", "ptpython"]
default_requirements = "$VMR_CONFIG_HOME/requirements.txt"
```



## Reference

### Commands

#### create
Create a virtual environment

#### activate
activate a virtual environment

#### deactivate
deactivate the currently active virtual environment

#### rm
delete a virtual environment

#### list
list all available virtual environments

#### healthcheck
make a healthcheck of the config and print a summary.

#### tmpenv
create a temporary virtual env (unnamed)
garbage collect stale data

#### inherit
create a virtual environment, that inherits from an existing one, thus it uses the parents package 'site-packages'

#### copy
copy a virtual environment (not sure of usability)

#### shellenv
print the necessary lines that need to be sourced by a users shell

#### cdenv
change into the directory of a virtual environment

#### path
print the path of a virtual environment

#### lsenv
list the contents of a virtual environments site-packages subfolder

#### lslinks
list linked projects of a virtual environment (this options would need the mapping in plain text not hash, but nothing
speaks against this?).

#### linkedenvs
list all linked virtual environments to a path

### local path resolution and linking
If a command is utilized using the local path and using it to link or fetch linking information the path will resolved
as follows.

It will search from `.` its parents until it either finds a (in this order) `pyproject.toml`, `setup.cfg`, `setup.py` or `.git` and use
the folder containing the found item. If nothing is found until reaching `~` then the current folder will be used.


### Interactive

There are interactive versions of commands, they are meant to be used with aliases, much inspired by
[forgit](https://github.com/wfxr/forgit).

They can be opted in or out, as you might want to choose your aliases.

Their full commands are shell functions names 
`vmr_i_$function` with the 'i' for interactive.

for these we have aliases, that utilize a 'fuzzyfinder ui' and show you dropdown menus or other interactive niceties.

`vma` - vmr activate
`vmc` - vmr create
`vml` - vmr list
`vmd` - vmr deactivate (just a shell alias)
`vmrm` - vmr rm
`vmcp` - vmr copy
`vmi` - vmr inherit


### Config



Configuration can be global or per command. Environment variables or CLI flags that change behavior are also mentioned.

Some options are global but do not make sense for each command; this is not mentioned here.

| Command    | Config          | Environment          | CLI              | Default                              | Description                                          |
|------------|-----------------|----------------------|------------------|--------------------------------------|------------------------------------------------------|
| create     | backend         | VMR_BACKEND          | --backend        | uv                                   | The backend to use for creating virtual environments |
| activate   | venv_path       | VMR_VENV_PATH        | --venv-path      | $XDG_DATA_HOME/vmr/venvs             | The path to the virtual environments                 |
| rm         | force           | VMR_RM_FORCE         | --force          | false                                | Force deletion without confirmation                  |
| list       | show_paths      | VMR_LIST_SHOW_PATHS  | --show-paths     | false                                | Show paths in the list output                        |
| healthcheck|                 |                      |                  |                                      |                                                      |
| tmpenv     | tmp_path        | VMR_TMP_PATH         | --tmp-path       | /tmp                                 | Path for temporary environments                      |
| gc         | retention_days  | VMR_GC_RETENTION_DAYS| --retention-days | 30                                   | Number of days to retain environments                |
| inherit    | base_venv       | VMR_INHERIT_BASE     | --base-venv      |                                      | The base virtual environment to inherit from         |
| copy       |                 |                      |                  |                                      |                                                      |
| shellenv   |                 |                      |                  |                                      |                                                      |
| cdenv      |                 |                      |                  |                                      |                                                      |
| path       |                 |                      |                  |                                      |                                                      |
| lsenv      |                 |                      |                  |                                      |                                                      |
| lslinks    |                 |                      |                  |                                      |                                                      |
| linkedenvs |                 |                      |                  |                                      |                                                      |

## Issues

I am happy for any adaption and support for this tool. I will try to solve issues when I can.

I will thus **NOT** take care of the following type of issues:
- support for Windows 
- support for MacOS
I am not using either of these operating systems nor am I interested into digging into their use, or create test
environments for them. Thus if no one offers a 'PR' for such issues, I will close these after some time.

## Contributing


## Versioning

I use [CalVer](https://calver.org/) for versioning.
Basically each new version will drop support of a previous version.
I will try to keep the API as stable as possible, deprecating features 'softly' with a 'WARNING' for a period until
deprecating 'hard'.
This is an application nothing should be dependent on, so 'CalVer' seems like a reasonable choice for me.

## Roadmap

- maybe rewrite in a more performant language than python - also to have it more isolated from python.
- support plugins, for different tools not supported by 'core'
- move supported tools from 'core' to plugins to have easier adaption to changes?


## Authors

* **Bertold Sedlak** 


## License

MIT


## Built with

- [README Template](https://gist.github.com/PurpleBooth/109311bb0361f32d87a2)
