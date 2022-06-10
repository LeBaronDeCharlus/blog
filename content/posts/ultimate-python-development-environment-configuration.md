---
title: "Ultimate Python development environment configuration"
date: 2022-06-10T08:12:12+02:00
image: /images/ultimate-python.png 
tags: [programming, direnv, virtualenv, secrets, python]
type: "post"
weight: 25
showTableOfContents: true
draft: false
---

![Ultimate Python](/images/ultimate-python.png)

1. [The Old Way](#the-old-way)
2. [Direnv](#direnv)
   - [What is direnv and how it works](#what-is-direnv-and-how-it-works)
   - [direnv installation](#direnv-installation)
   - [Shell configuration](#shell-configuration)
   - [How Python works with direnv](#how-python-works-with-direnv)
3. [Pyenv](#pyenv)
   - [Installation and Configuration](#installation-and-configuration)
   - [Use pyenv with direnv](#use-pyenv-with-direnv)
4. [GPG](#gpg)
   - [Generate a key](#generate-a-key)
   - [Get key ID](#get-key-id)
5. [Pass](#pass)
   - [Pass installation](#pass-installation)
   - [Pass usage](#pass-usage)
6. [Tomb](#tomb)
   - [Tomb installation](#tomb-installation)
   - [Tomb usage](#tomb-usage)
7. [Secrets with Direnv and Pass](#secrets-with-direnv-and-pass)
8. [Direnv advanced configurations](#direnv-advanced-configurations)
   - [Check for commands dependencies](#check-for-commands-dependencies)
   - [Third-party binaries](#third-party-binaries)
   - [Third-party configs](#third-party-configs)
11. [Final template](#final-template)
12. [Conclusion](#conclusion)

Disclaimer: I'm not saying `virtualenv` methods are bad.

# The Old way

For years now, I've been using Python for a lot of different projects. My setup environment was quite simple, use `virtualenv` and work locally.

But I've faced few pain points by doing so. I was often forgetting to `source venv/bin/activate`, meaning I `pip install my-package` on my global system. 

I also had bad secrets management by setting them not encrypted in a Shell script loaded at my program start.

As bad as:

```shell
export DB_NAME="db_name"
export DB_USER="user"
export DB_PASS="pass"
export DB_HOST=172.17.0.2
export DB_PORT=3306
```

Even if some great code editors can handle `python interpreters` in virtual environment, such as Vscode, I felt like my workflow automation was not as complete as it could be.

# Direnv

How to ensure not forgetting to load a virtual environment and install package globally while working on our project ?

### What is direnv and how it works

From official documentation

> direnv is an extension for your shell. It augments existing shells with a new feature that can load and unload environment variables depending on the current directory.
> Before each prompt, direnv checks for the existence of a .envrc file (and optionally a .env file) in the current and parent directories. If the file exists (and is authorized), it is loaded into a bash sub-shell and all exported variables are then captured by direnv and then made available to the current shell.
>
> It supports hooks for all the common shells like bash, zsh, tcsh and fish. This allows project-specific environment variables without cluttering the ~/.profile file.
>
> Because direnv is compiled into a single static executable, it is fast enough to be unnoticeable on each prompt. It is also language-agnostic and can be used to build solutions similar to rbenv, pyenv and phpenv.

### direnv installation

`direnv` is accessible through packages in almost all distributions.

#### Package installation

If you want a global system installation :
```shell
λ ~/ sudo apt-get install direnv
```

#### Manual installation

If you want custom installation, take a look on this [script](https://direnv.net/install.sh) hosted by direnv official documentation.

```shell
λ ~/ curl -sfL https://direnv.net/install.sh | bash
```

### Shell configuration

Once `direnv` is installed you need to configure your `$SHELL` in order to hook it with your default shell.
It supports `bash`, `zsh`, `fish`, `tcsh` and `elvish`.

_e.g with zsh_
```shell
λ ~/ echo 'eval "$(direnv hook zsh)"' >> ~/.zshrc
λ ~/ source ~/.zshrc
```

### How python works with direnv

We can load a Python virtual environment thanks to `direnv`, to do so we need to specify `layout` command in a `.envrc` file located at our root project.

Let's create our project directory first.

```shell
λ ~/ mkdir project && cd project
```

Creat our `.envrc` file.

```shell
λ ~/project/ echo 'layout python3' > .envrc
```

You will notice an error message like :
```
direnv: error project/.envrc is blocked. Run `direnv allow` to approve its content
```

This is a security way to block default content in your file. You can allow it thanks to :
```shell
λ ~/project/ direnv allow
direnv: loading ~/project/.envrc                                                                                                                                                                                     
direnv: export +VIRTUAL_ENV ~PATH
```

Output inform us that virtual environment was automatically created for us. You won't see any prompt modification (as `virtualenv` does while `source venv/bin/activate`) it's normal.

Quick check of our Python binary with :
```shell
λ ~/project/ which python
project/.direnv/python-3.10.4/bin/python
```

You can then work as you normally do through `venv` by installing your python dependencies.
```shell
# install a single package
λ ~/project/ pip install django

# or install your project dependencies
λ ~/project/ pip install -r requirements.txt
```

Just to be clear, `direnv` will automatically load your virtual environement when you move in your project directory. As soon as you will move out, environment will also be automatically deactivated. 
```shell
λ ~/project/ cd 
direnv: unloading
```

That's could be enough for common use if you don't need a specific Python version other than one installed on your system, in some cases you will want to work with specific versions, that's why we need `pyenv`.

# Pyenv

`pyenv` lets you easily switch between multiple versions of Python. It's simple, unobtrusive, and follows the UNIX tradition of single-purpose tools that do one thing well. 
It allows you to change Python version for each project and it supported by `direnv` since `2.21.0`.

## Installation and Configuration

Get `pyenv` :
```shell
λ ~/ curl -L https://pyenv.run | bash
```

Configure your $SHELL.

_e.g with zsh_
```shell
λ ~/ echo 'export PYENV_ROOT="$HOME/.pyenv"' >> ~/.zshrc
λ ~/ echo 'command -v pyenv >/dev/null || export PATH="$PYENV_ROOT/bin:$PATH"' >> ~/.zshrc
λ ~/ echo 'eval "$(pyenv init -)"' >> ~/.zshrc 
```

Check installation:
```shell
λ ~/ pyenv --version
pyenv 2.3.1
```

And install latest Python version at this date `3.10.5`:
```shell
λ ~/ pyenv install 3.10.5
```

## Use pyenv with direnv

Now `pyenv` is installed, let's interface it with our project and `direnv`.
```shell
λ ~/ cd project/
λ ~/ echo 'layout pyenv 3.10.5' > .envrc
λ ~/ direnv allow
```

Check our Python version and interpreter :
```shell
λ ~/project/ python --version && which python
Python 3.10.5
~/project/.direnv/python-3.10.5/bin/python
```

That's it.


# GPG

Remember about managing secrets ? Now that we have our virtual environements set, how can we manage to work with secrets without write them directly in our project/repository ?

Before we can use a password manager such as `pass` we need a `gpg key id`.

## Generate a key
Let's first generete one.

```shell
λ ~/ gpg --full-generate-key

Please select what kind of key you want:
   (1) RSA and RSA (default)
   (2) DSA and Elgamal
   (3) DSA (sign only)
   (4) RSA (sign only)
  (14) Existing key from card
Your selection? 1

RSA keys may be between 1024 and 4096 bits long.
What keysize do you want? (3072) 
Requested keysize is 3072 bits
Please specify how long the key should be valid.
         0 = key does not expire
      <n>  = key expires in n days
      <n>w = key expires in n weeks
      <n>m = key expires in n months
      <n>y = key expires in n years

Key is valid for? (0) 
Key does not expire at all
Is this correct? (y/N) y

Real name: Kader Ovski
Email address: me@kaderovski.com
Comment: 
You selected this USER-ID:
    "Kader Ovski <me@kaderovski.com>"

Change (N)ame, (C)omment, (E)mail or (O)kay/(Q)uit? O

We need to generate a lot of random bytes. It is a good idea to perform
some other action (type on the keyboard, move the mouse, utilize the
disks) during the prime generation; this gives the random number
generator a better chance to gain enough entropy.

gpg: key 0854057891EFB8F0 marked as ultimately trusted

pub   rsa3072 2022-06-10 [SC]
      DC3EC748A8D97169F47C16690854057891EFB8F0
uid                      Kader Ovski <me@kaderovski.com>
sub   rsa3072 2022-06-10 [E]
```

## Get key ID
We can then list our new key :
```shell
λ ~/ gpg --list-keys
gpg: checking the trustdb
gpg: marginals needed: 3  completes needed: 1  trust model: pgp
gpg: depth: 0  valid:   1  signed:   0  trust: 0-, 0q, 0n, 0m, 0f, 1u
$HOME/.gnupg/pubring.kbx
------------------------
pub   rsa3072 2022-06-10 [SC]
      DC3EC748A8D97169F47C16690854057891EFB8F0
uid           [ultimate] Kader Ovski <me@kaderovski.com>
sub   rsa3072 2022-06-10 [E]
```

# Pass

`pass` is the standard unix password manager.

> With pass, each password lives inside of a gpg encrypted file whose filename is the title of the website or resource that requires the password. These encrypted files may be organized into meaningful folder hierarchies, copied from computer to computer, and, in general, manipulated using standard command line file management utilities.

## Pass installation

`pass` is available in almost all distribution and can be installed as follow:
```shell
λ ~/ sudo apt-get install pass
```

## Pass usage

Now that we have `pass` installed on our system **and** a valid `gpg_key_id` we can init our password manager with : 
```shell
λ ~/ pass init DC3EC748A8D97169F47C16690854057891EFB8F0
mkdir: created directory '$HOME/.password-store/'
Password store initialized for DC3EC748A8D97169F47C16690854057891EFB8F0
```

Let's insert our first password :
```shell
λ ~/ pass insert kaderovski.com/blog/some_secret
mkdir: created directory '$HOME/.password-store/kaderovski.com'
mkdir: created directory '$HOME/.password-store/kaderovski.com/blog'
Enter password for kaderovski.com/blog/some_secret: 
Retype password for kaderovski.com/blog/some_secret: 
```

Our password is now store in our `pass` :
```shell
λ ~/ pass
Password Store
`-- kaderovski.com
    `-- blog
        `-- some_secret
```

You can get your secret by typing and entering your gpg secret passphrase :
```shell
λ ~/ pass show kaderovski.com/blog/some_secret
my_secret
```

# Tomb

Due to the structure of pass, file and directory names are not encrypted in the password store. `pass-tomb` provides a convenient solution to put your password store in a Tomb and then keep your password tree encrypted when you are not using it.

It uses the same GPG key to encrypt passwords and tomb, therefore you don't need to manage more key or secret. Moreover, you can ask pass-tomb to automatically close your store after a given time.

## Tomb installation

You can use package installation on your distribution :
```shell
λ ~/ sudo apt install pass-extension-tomb
```

## Tomb usage
Create a new password tomb:

_e.g_
```shell
> pass tomb DC3EC748A8D97169F47C16690854057891EFB8F0
 (*) Your password tomb has been created and opened in ~/.password-store.
 (*) Password store initialized for DC3EC748A8D97169F47C16690854057891EFB8F0
  .  Your tomb is: ~/.password.tomb
  .  Your tomb key is: ~/.password.key.tomb
  .  You can now use pass as usual.
  .  When finished, close the password tomb using 'pass close'.
```

You can now use the best part of `pass-tomb`:

- open your tomb
```shell
λ ~/ pass open
 (*) Your password tomb has been opened in /home/kaderovski/.password-store/.
  .  You can now use pass as usual.
  .  When finished, close the password tomb using 'pass close'.
```

- close your tomb
```shell
λ ~/ pass close
 (*) Your password tomb has been closed.
  .  Your passwords remain present in /home/kaderovski/.password.tomb.
```

Let's check what is happening with `open` and `close` options.

When our `tomb` is closed, `pass` command and our passwords path looks like this:
```shell
λ ~/ pass
Password Store
└── 2

λ ~/ tree .password-store/            
.password-store/
└── 2

1 directory, 0 files
```

We can not see our key:value (path/password-name) elements on output and filesystem.

When opening our `tomb` we are decrypting our password database :
```shell
λ ~/ pass ; tree .password-store/
Password Store
├── kaderovski.com
│   └── blog
│       └── some_secret.gpg

λ ~/ tree .password-store/            
.password-store/
├── kaderovski.com
│   └── blog
│       └── some_secret.gpg
```

# Secrets with direnv and pass
We can now manage to use our `secrets` in our Python code thanks to `direnv` and `pass`.

We need to modify a bit our `.envrc` in our project directory by checking if the `tomb` is open or not.
I'm doing it by checking if I can `stat` my `$HOME/.password-store/.gpg-id` which is possible when the `tomb` is open. If not it means `tomb` is close and need to be open. 

```shell
# .envrc

layout pyenv 3.10.5

# CHEKING IF TOMB IS OPEN
if ! stat $HOME/.password-store/.gpg-id > /dev/null 2>&1 ; then
    # IF NOT OPEN IT
    pass open
fi

# THEN PUT SECRET IN ENV
export MY_SECRET=$(pass show kaderovski.com/blog/some_secret)
```

We can now reload our `direnv`.
```shell
λ ~/project/ direnv reload
direnv: loading ~/project/.envrc
 (*) Your password tomb has been opened in /home/$USER/.password-store/.
  .  You can now use pass as usual.
  .  When finished, close the password tomb using 'pass close'.
direnv: export +MY_SECRET +VIRTUAL_ENV ~PATH
```

Notice here `+MY_SECRET` in the output, telling us it is accessible through our environment.

We can try to reach our secret :
```shell
λ ~/project/ echo $MY_SECRET
my_secret
```

What is really important here is that **we are not storing clear secrets** in our code, project or repo.
All our secrets will be reachable through our virtual environment and can be used in you Python code like this :
```python
>>> import os
>>> secret=os.environ.get('MY_SECRET')
>>> print(secret)
my_secret
```
# Direnv advanced configurations

In more complex projects you may need `commands`, `external dependencies`, `third-party`, or `third-party configuration`. Let's see how we can manage them with `direnv`.

## Check for commands dependencies
In your `.envrc` file, you can ensure you can use and reach some commands needed in your project.

You can do as follow:
```shell
# .envrc

layout pyenv 3.10.5

# CHEKING IF TOMB IS OPEN
if ! stat $HOME/.password-store/.gpg-id > /dev/null 2>&1 ; then
    # IF NOT OPEN IT
    pass open
fi

# THEN PUT SECRET IN ENV
export MY_SECRET=$(pass show kaderovski.com/blog/some_secret)

# CHECKING COMMANDS DEPENDENCIES
DIRENV_CMD_DEPENDENCIES="unzip tar mkdir curl chmod rm"
for mandatory_cmd in ${DIRENV_CMD_DEPENDENCIES}; do
   if [ -z "$(which ${mandatory_cmd})" ]; then
        echo "===> Mandatory command not found: ${mandatory_cmd}"
        exit 1
   fi
done
```

## Third-party binaries

We can arrange to use specific third-party binaries such as `packer`, `terraform`, `vault` etc…

We will tell `.direnv` to use `PATH` in order to place our third-party binaries in `.direnv/bin` path.

Let's grab our `.envrc` file again:
```shell
# .envrc

layout pyenv 3.10.5

# CHEKING IF TOMB IS OPEN
if ! stat $HOME/.password-store/.gpg-id > /dev/null 2>&1 ; then
    # IF NOT OPEN IT
    pass open
fi

# THEN PUT SECRET IN ENV
export MY_SECRET=$(pass show kaderovski.com/blog/some_secret)

# CHECKING COMMANDS DEPENDENCIES
DIRENV_CMD_DEPENDENCIES="unzip tar mkdir curl chmod rm"
for mandatory_cmd in ${DIRENV_CMD_DEPENDENCIES}; do
   if [ -z "$(which ${mandatory_cmd})" ]; then
        echo "===> Mandatory command not found: ${mandatory_cmd}"
        exit 1
   fi
done

export DIRENV_TMP_DIR="${PWD}/.direnv"
export DIRENV_BIN_DIR="${DIRENV_TMP_DIR}/bin"
if [ ! -e "${DIRENV_BIN_DIR}" ]; then
   mkdir -p "${DIRENV_BIN_DIR}"
fi
export PATH="${DIRENV_BIN_DIR}:${PATH}"
```

Now install our binary, let's try it with `packer` Hashicorp binary :

```shell
# .envrc

layout pyenv 3.10.5

# CHEKING IF TOMB IS OPEN
if ! stat $HOME/.password-store/.gpg-id > /dev/null 2>&1 ; then
    # IF NOT OPEN IT
    pass open
fi

# THEN PUT SECRET IN ENV
export MY_SECRET=$(pass show kaderovski.com/blog/some_secret)

# CHECKING COMMANDS DEPENDENCIES
DIRENV_CMD_DEPENDENCIES="unzip tar mkdir curl chmod rm"
for mandatory_cmd in ${DIRENV_CMD_DEPENDENCIES}; do
   if [ -z "$(which ${mandatory_cmd})" ]; then
        echo "===> Mandatory command not found: ${mandatory_cmd}"
        exit 1
   fi
done

export DIRENV_TMP_DIR="${PWD}/.direnv"
export DIRENV_BIN_DIR="${DIRENV_TMP_DIR}/bin"
if [ ! -e "${DIRENV_BIN_DIR}" ]; then
   mkdir -p "${DIRENV_BIN_DIR}"
fi
export PATH="${DIRENV_BIN_DIR}:${PATH}"

# PACKER INSTALLATION

PACKER_VERSION="1.8.1"
PACKER_ARCH="linux_amd64"
PACKER_PKG_NAME="packer_${PACKER_VERSION}_${PACKER_ARCH}.zip"
PACKER_PKG_URL="https://releases.hashicorp.com/packer/${PACKER_VERSION}/${PACKER_PKG_NAME}"
PACKER_PKG_PATH="${DIRENV_TMP_DIR}/${PACKER_PKG_NAME}"
if [ ! -e "${DIRENV_BIN_DIR}/packer" ]; then
   echo "===> Getting packer:${PACKER_VERSION}:${PACKER_ARCH} (can take a while to execute)"
   curl -s -L "${PACKER_PKG_URL}" -o "${PACKER_PKG_PATH}"
   unzip ${PACKER_PKG_PATH} -d ${DIRENV_BIN_DIR}
   chmod 700 ${DIRENV_BIN_DIR}/packer
   rm -f ${PACKER_PKG_PATH}
fi
```

Let's try out our new configuration :
```shell
λ ~/project direnv allow
direnv: loading ~/project/.envrc                                                                    
===> Getting packer:1.8.1:linux_amd64 (can take a while to execute)
Archive:  ~/project/.direnv/packer_1.8.1_linux_amd64.zip
  inflating: ~/project/.direnv/bin/packer  
direnv: export +MY_SECRET +VIRTUAL_ENV ~PATH
```

Now `packer` is installed in our `PATH` let's locate it and run it :
```shell
λ ~/project/ which packer
~/project/.direnv/bin/packer
λ ~/ packer --version
1.8.1
```

If in the futur you need to change `packer` version you just have to remove your current `packer` binary and modify `PACKER_VERSION` variable to rebuild your `direnv`.

## Third-party configs

Of course, if our third-party need custom (or not) configurations we can specify them in our `.envrc` but to keep pour code clear and organized we also can split our configs into subfiles.

By adding this block at the end of our `.envrc` file :
```shell
# .envrc 

# […]

# ADDONS
ENV_ADDONS=".env.packer .env.custom_config"
for addon in ${ENV_ADDONS}; do
   if [ -e "${PWD}/${addon}" ]; then
       source ${PWD}/${addon}
   fi
done
```

And then create your addons custom config file :
```shell
# .env.packer
export PACKER_VAR1=VAR1_VALUE
export PACKER_VAR2=VAR2_VALUE
```

```shell
# .env.custom_config
export CUSTOM_VAR1=CUSTOM1_VALUE
export CUSTOM_VAR2=CUSTOM2_VALUE
```

Don't forget to reload:
```shell
λ ~/project/ direnv reload
```

# Final template

Here is a final `.envrc` template you can grab and edit for your needs :
```shell
# .envrc

layout pyenv 3.10.5

# CHEKING IF TOMB IS OPEN
if ! stat $HOME/.password-store/.gpg-id > /dev/null 2>&1 ; then
    # IF NOT OPEN IT
    pass open
fi

# THEN PUT SECRET IN ENV
export MY_SECRET=$(pass show kaderovski.com/blog/some_secret)

# CHECKING COMMANDS DEPENDENCIES
DIRENV_CMD_DEPENDENCIES="unzip tar mkdir curl chmod rm"
for mandatory_cmd in ${DIRENV_CMD_DEPENDENCIES}; do
   if [ -z "$(which ${mandatory_cmd})" ]; then
        echo "===> Mandatory command not found: ${mandatory_cmd}"
        exit 1
   fi
done

export DIRENV_TMP_DIR="${PWD}/.direnv"
export DIRENV_BIN_DIR="${DIRENV_TMP_DIR}/bin"
if [ ! -e "${DIRENV_BIN_DIR}" ]; then
   mkdir -p "${DIRENV_BIN_DIR}"
fi
export PATH="${DIRENV_BIN_DIR}:${PATH}"

# PACKER INSTALLATION

PACKER_VERSION="1.8.1"
PACKER_ARCH="linux_amd64"
PACKER_PKG_NAME="packer_${PACKER_VERSION}_${PACKER_ARCH}.zip"
PACKER_PKG_URL="https://releases.hashicorp.com/packer/${PACKER_VERSION}/${PACKER_PKG_NAME}"
PACKER_PKG_PATH="${DIRENV_TMP_DIR}/${PACKER_PKG_NAME}"
if [ ! -e "${DIRENV_BIN_DIR}/packer" ]; then
   echo "===> Getting packer:${PACKER_VERSION}:${PACKER_ARCH} (can take a while to execute)"
   curl -s -L "${PACKER_PKG_URL}" -o "${PACKER_PKG_PATH}"
   unzip ${PACKER_PKG_PATH} -d ${DIRENV_BIN_DIR}
   chmod 700 ${DIRENV_BIN_DIR}/packer
   rm -f ${PACKER_PKG_PATH}
fi

# ADDONS
ENV_ADDONS=".env.packer .env.custom_config"
for addon in ${ENV_ADDONS}; do
   if [ -e "${PWD}/${addon}" ]; then
       source ${PWD}/${addon}
   fi
done
```

# Conclusion

Yes this require a bit of knowledge and a some configurations to be efficient in this workflow, but keep in mind that all will be configured automatically and will be more easier version controlled.

You can arrange in order to put your application in production area to share your GPG key or to manage multiple GPG key in a same `pass` (_eg one GPG ID for each team member_), version your pass and make it a git version accessible.

All dependencies, with the same packages will be deployed exactly as your developpment area.

Keep it simple.
