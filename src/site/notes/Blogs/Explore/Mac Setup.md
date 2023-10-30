---
{"dg-publish":true,"permalink":"/blogs/explore/mac-setup/","tags":["todo","blogs"],"created":"2023-09-13T21:59:39.000+08:00","updated":"2023-10-31T00:08:05.383+08:00"}
---

# Enhance Your Mac Experience with a Custom Setup Script

Are you a Mac user who loves to customize their setup? Well, you're in luck because today I'm going to introduce you to a fantastic Mac setup script that will **save you time and effort**. With just a few commands, you can install some of the most popular tools and applications for your Mac, including **brew**, **ohmyzsh**, **nvim**, **yabai**, and even clone your config files from GitHub. So let's dive right in and see how this script can level up your Mac experience!

## Brew: The Ultimate Package Manager for macOS

First things first, let's talk about brew. If you're not familiar with it, brew is a package manager for macOS that allows you to easily install command-line tools and applications. It's like an App Store for developers! With brew installed on your Mac, you can quickly add or remove packages with simple commands.

To get started with the setup script, open up your Terminal application (you can find it in the Utilities folder within Applications) and run the following command:

```bash
/bin/bash -c "$(curl -fsSL https://raw.githubusercontent.com/Homebrew/install/HEAD/install.sh)"
```

This command will download and install brew on your system. Once it's done, you'll have access to thousands of packages that will enhance your Mac experience.

## Ohmyzsh: Customize Your Terminal Configuration

Next up is ohmyzsh. Ohmyzsh is a delightful open-source framework for managing your zsh configuration. It comes with tons of plugins and themes that make working with the terminal a breeze. To install ohmyzsh using brew, simply run the following command:

```bash
brew install oh-my-zsh
```

Once installed, you can customize your zsh configuration by editing the `.zshrc` file located in your home directory.

## Nvim: The Next Generation Text Editor

Now it's time to set up nvim (or Neovim), which is an improved version of Vim. It provides better performance and additional features for developers who love working in the terminal. To install nvim using brew, run the following command:

```bash
brew install neovim
```

Once installed, you can start using nvim by running `nvim` in your Terminal. You can also customize nvim's configuration by editing the `init.vim` file located in `~/.config/nvim/`.


## Yabai: Take Control of Your Windows

Yabai is a tiling window manager for macOS that allows you to organize your windows efficiently. It gives you full control over window placement, resizing, and more. To install yabai using brew, run the following command in your terminal:

```bash
brew install koekeishiya/formulae/yabai
```

Once yabai is installed, you will need to enable the accessibility permissions for it to work properly. To do this, go to "System Preferences" > "Security & Privacy" > "Privacy" > "Accessibility" and check the box next to yabai.

To start yabai automatically on system startup, you can add it as a login item. Go to "System Preferences" > "Users & Groups" > "Login Items" and click on the "+" button. Navigate to the yabai binary (usually located at /usr/local/bin/yabai) and add it to the login items.

To configure yabai, you can create a configuration file called ".yabairc" in your home directory. This file should contain the settings and keybindings for yabai. You can find example configuration files online or refer to the official documentation for more information on how to customize yabai.

Once you have set up yabai, you can use various keybindings and commands to manage your windows. For example, pressing "alt + space" will activate the window switcher, allowing you to quickly switch between windows. Pressing "alt + shift + j/k/h/l" will move focus between different windows, while pressing "alt + shift + enter" will swap the focused window with the largest window in the current space.

Overall, yabai is a powerful tool for organizing and managing windows on macOS, providing a more efficient workflow for multitasking.



Note: Before running any commands or scripts, it's always a good idea to backup your important files and make sure you understand what the commands do.

## Cloning Config Files from GitHub

If you have your config files (such as `.zshrc`, `init.vim`, and `.yabairc`) stored in a GitHub repository, you can use the following command to clone them directly onto your Mac:

```bash
git clone <repository_url> ~/config-files
```

Replace `<repository_url>` with the URL of your repository. This command will clone the repository into a folder called `config-files` in your home directory.

## Automating the Setup Process

To automate the setup process, we can create a custom script that installs brew, ohmyzsh, nvim, and yabai, and then clones the config files from GitHub. Here's an example of how such a script could look like:

```bash
#!/bin/bash

# Install brew
/bin/bash -c "$(curl -fsSL https://raw.githubusercontent.com/Homebrew/install/HEAD/install.sh)"

# Install ohmyzsh
brew install oh-my-zsh

# Install nvim
brew install neovim

# Install yabai
brew install koekeishiya/formulae/yabai

# Enable accessibility permissions for yabai
osascript -e 'tell application "System Events" to tell process "Security & Privacy" to tell window 1 to click checkbox 1 of row 2 of table 1 of scroll area 1'

# Clone config files from GitHub
git clone <repository_url> ~/config-files

# Copy config files to their respective locations
cp ~/config-files/.zshrc ~/.zshrc
cp ~/config-files/init.vim ~/.config/nvim/init.vim
cp ~/config-files/.yabairc ~/.yabairc

# Clean up
rm -rf ~/config-files

echo "Setup complete!"
```

Replace `<repository_url>` with the URL of your repository.

Save the script to a file (e.g., `mac_setup.sh`) and make it executable by running `chmod +x mac_setup.sh` in the Terminal. Then, you can run the script by executing `./mac_setup.sh`. The script will install brew, ohmyzsh, nvim, and yabai, enable accessibility permissions for yabai, clone your config files from GitHub, and

In conclusion, by using this Mac setup script, you can easily install and configure brew, ohmyzsh, nvim, and yabai to enhance your Mac experience. These tools will help you customize your terminal, improve your text editing capabilities, and organize your windows more efficiently. So why wait? Give this script a try and take your Mac setup to the next level!