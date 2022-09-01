# Menginstall ZSH Suggestion di Linux Ubuntu

<a href="https://asciinema.org/a/37390" target="_blank"><img src="https://asciinema.org/a/37390.png" width="400" /></a>

1. Clone this repository somewhere on your machine. This guide will assume ~/.zsh/zsh-autosuggestions.

    ```git clone https://github.com/zsh-users/zsh-autosuggestions ~/.zsh/zsh-autosuggestions```

2. Add the following to your .zshrc:

    ```source ~/.zsh/zsh-autosuggestions/zsh-autosuggestions.zsh```

3. Start a new terminal session.

## Uninstallation

1. Remove the code referencing this plugin from ~/.zshrc.
2. Remove the git repository from your hard drive

    ```rm -rf ~/.zsh/zsh-autosuggestions # Or wherever you installed```
