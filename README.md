# Just For Practice

Try to make really working [Git](http://git-scm.com/) implementation for [Sublime Text 3](http://www.sublimetext.com/3) on [Windows 7 64bit](http://windows.microsoft.com/en-us/windows/windows-help#windows=windows-7) which will work with SSH keys on [GitHub](https://github.com)

## Plugins to try

1. 	Sublime Text [Git](https://github.com/kemayo/sublime-text-git): Git integration: it's pretty handy. Who knew, right?
2. 	[SideBarGit](https://github.com/titoBouzout/SideBarGit): Add git commands to sidebar. Textual port of komodin extension for sublime text.

## HOWTO's

1. 	[Using Git Inside of Sublime Text to Improve Workflow](https://scotch.io/tutorials/using-git-inside-of-sublime-text-to-improve-workflow) by Chris Sevilleja ([@sevilayha](https://twitter.com/sevilayha))

*~ Aleksandar Urosevic*

## Step by Step

* 	[Working with SSH key passphrases](https://help.github.com/articles/working-with-ssh-key-passphrases/)
* 	[SublimeGit issues](https://github.com/SublimeGit/SublimeGit/issues/3)

1. 	Create ppk file and store it to `C:\Users\aleksandar\.ssh\id_rsa` or use your existing key file.
2. 	Run **Git Bash**.
3. 	Execute `ssh-agent`
4. 	Execute `ssh-add` if you have `id_rsa` private key, or if you have it with different name then `~/.ssh/id_rsa_github`, and enter passphrase.
5. 	Create file **C:\Users\aleksandar\.bashrc** and add following content to it:

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

**NOTE:** If you have key stored in file with different name than `id_rsa`, replace lines `ssh-add` with `ssh-add ~/.ssh/id_rsa_github` in `.bashrc` file.

6. Close **Git Bash** and run **Sublime Text 3**
7. Now you should use SublimeGit Fetch/Pull/Pus w/o problem.