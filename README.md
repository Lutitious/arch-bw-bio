## Install

First remove any existing bitwarden instances using pacman

`sudo pacman -Rcns bitwarden-desktop`

Next clone the repo and install via makepkg

```
git clone https://github.com/Lutitious/arch-bw-bio
cd arch-bw-bio
makepkg -si --skipinteg
```
## Browser Integration
If you care about browser integration log in and follow these steps

1) Go to File > Settings
2) Check the box `Unlock with polkit`
3) Scroll down and enable `Allow browser integration` and optionally `Require verification for browser integration`
4) Go to `~/.mozilla/native-messaging-hosts` on Firefox or `~/.config/google-chrome/NativeMessagingHosts/` on Chrome
5) Open the json and make sure the PATH is set to `/usr/bin/bitwarden-desktop`. If not, change it to that and optionally set the file to read only.
