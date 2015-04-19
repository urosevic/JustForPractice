# SublimeGit and Windows 7 (Just For Practice)

Make really working [Git](http://git-scm.com/) implementation for [Sublime Text 3](http://www.sublimetext.com/3) on [Windows 7 64bit](http://windows.microsoft.com/en-us/windows/windows-help#windows=windows-7) which will work with OpenSSH SSH keys on [GitHub](https://github.com) and [GitLab](https://about.gitlab.com/).

All SSH files are stored on path `C:\Users\aleksandar\.ssh\` (just replace *aleksandar* with your own username).

Check out my [Sublime Text cheatsheet](https://github.com/urosevic/sublime).

![ScreenShot](/screenshots/JustForPractice.png)

## Sublime Text plugins to try

1. 	[SublimeGit](https://sublimegit.net/) **Additional manual tweaks required to make remote repos to work**
2. 	Sublime Text [Git](https://github.com/kemayo/sublime-text-git): Git integration: it's pretty handy. Who knew, right? **Same as above with remotes**
3. 	[SideBarGit](https://github.com/titoBouzout/SideBarGit): Add git commands to sidebar. Textual port of komodin extension for sublime text. **Works out of the box with remotes, but asks for passphrase on every interaction with remote repo.**

## Generate SSH Private and Public Key on Windows (if you don't have key already)

1. 	If you don't have already installed Putty, you can download only [PuTTYgen](http://www.chiark.greenend.org.uk/~sgtatham/putty/download.html) and run that program.
2. 	Select **SSH-2 RSA** option in **Parameters** section and click button **Generate**.
3. 	Move mouse cursor over PuTTYgen window to generate some randomness.
4. 	When key is generated, add good comment (short explanation who is owner of key and for what you'll use it, for example).
5. 	Set *Key passphrase* (password) which you'll use to identify yourself when some app ro service need to use your key. Set good password, and remember it!
6. 	Select content in **Key** area (begins with *ssh-rsa* and end with your *Key comment*) and save to some file, for example `id_rsa.ssh-rsa` located at path `C:\Users\aleksandar\.ssh\`. This is your private key and you'll use it on *GitHub* or *GitLab* credential configuration.
7. 	From **Conversion** menu select option **Export OpenSSH Key** and save file to same directory as above, but with name `is_rsa` (w/o extension). This is your private key (**never share it with others**), and it will be used for **Git** shell.
8. 	Optionally, to reuse your private key latter with PuTTY or other apps, save it in PuTTY format:
 1. Click **Save public key** and save public key file with name `is_rsa.pub`.
 2. Click **Save private key** and save private key file with name `is_rsa.ppk`.

## Install and Configure Git

### Step 1
Download [Git for Windows](http://git-scm.com/) and install it. You can leave all options on default, but make sure you select option **Run Git from the Windows Command Prompt** in second step.

### Step 2
You can use your existing SSH Key. If you don't have one, create new by following previous section. Make sure that your private **OpenSSH Key** is located at `C:\Users\aleksandar\.ssh\id_rsa`.

### Step 3
If there is no already existing, create new file `C:\Users\aleksandar\.ssh\config` and add to it configuration for remote Git repositories that you'll use. For example:

```
Host github.com
    Hostname github.com
    User urosevic
    PreferredAuthentications publickey
    IdentityFile C:\Users\aleksandar\.ssh\id_rsa
    TCPKeepAlive yes
    IdentitiesOnly yes
```

### Step 4
Run **Git Bash** from **Start** menu. This is app installed by Git for Windows.

### Step 5
Type and run command `ssh-agent` in **Git Bash** console.

### Step 6
If you named private OpenSSH Key as `id_rsa` only, the type and run command `ssh-add` w/o any parameter.
Othervise, if you saved private OpenSSH Key under different name like `open_ssh_github`, then type and run command `ssh-add ~/.ssh/id_rsa_github`.

You'll be asked for *passphrase* for your key, so type it. This will create `~/.ssh/agent.env` file used latter in script below.

### Step 7
If does not exists already, create new file `C:\Users\aleksandar\.bashrc` (just under your username) and add following content to it:

```
# Note: ~/.ssh/environment should not be used, as it
#       already has a different purpose in SSH.

env=~/.ssh/agent.env

# Note: Don't bother checking SSH_AGENT_PID. It's not used
#       by SSH itself, and it might even be incorrect
#       (for example, when using agent-forwarding over SSH).

agent_is_running() {
    if [ "$SSH_AUTH_SOCK" ]; then
        # ssh-add returns:
        #   0 = agent running, has keys
        #   1 = agent running, no keys
        #   2 = agent not running
        ssh-add -l >/dev/null 2>&1 || [ $? -eq 1 ]
    else
        false
    fi
}

agent_has_keys() {
    ssh-add -l >/dev/null 2>&1
}

agent_load_env() {
    . "$env" >/dev/null
}

agent_start() {
    (umask 077; ssh-agent >"$env")
    . "$env" >/dev/null
}

if ! agent_is_running; then
    agent_load_env
fi

# if your keys are not stored in ~/.ssh/id_rsa or ~/.ssh/id_dsa, you'll need
# to paste the proper path after ssh-add
if ! agent_is_running; then
    agent_start
    ssh-add
elif ! agent_has_keys; then
    ssh-add
fi

setx SSH_AUTH_SOCK $SSH_AUTH_SOCK 1> nul
setx SSH_AGENT_PID $SSH_AGENT_PID 1> nul

unset env
```

### Step 8
Close **Git Bash** if you don't need it anymore, and run **Sublime Text 3**.

### Step 9
Now you should use **SublimeGit** Fetch/Pull/Push and other actions with remotes w/o any problem.

Full process described above you'll do only once. On every next system boot (restart), you just run **Git Bash** (or add shortcut to **Startup** section in **Start** menu to do this automatically after login) and type OpenSSH Key passphrase. After that, you can close **Git Bash** and use SublimeGit w/o typing passphrase on every interaction with remotes.

* ~ Aleksandar Urosevic @20150419 *

## Credits

To make this tutorial I spent lot of time on Google and read a lot articles, but most useful was:
* [Using Git Inside of Sublime Text to Improve Workflow](https://scotch.io/tutorials/using-git-inside-of-sublime-text-to-improve-workflow) by Chris Sevilleja ([@sevilayha](https://twitter.com/sevilayha))
* [Working with SSH key passphrases](https://help.github.com/articles/working-with-ssh-key-passphrases/)
* [SublimeGit Remote Issues](https://docs.sublimegit.net/troubleshooting.html#remote-issues)
* [GitHub: SublimeGit Issues](https://github.com/SublimeGit/SublimeGit/issues/3)