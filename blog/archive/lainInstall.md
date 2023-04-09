# Making linux login beautiful with sddm and lain

Customization potential on linux has always awed me. It has always been a favourite passtime of mine to make my setup look like a retro system or a sci-fi system.

Few days ago while browsing the web I stumbled upon [this repo](https://gitlab.com/mixedCase/sddm-lain-wired-theme)  showcasing a sddm login screen inspired by 1998 anime ["Serial Experiments Lain"](https://myanimelist.net/anime/339/Serial_Experiments_Lain).

I was ecstatic,I like that show to the point of using lain as my profile pic across various platforms. 
I needed to have that on my laptop asap.

I installed the theme and when trying it out I found that all the elements were way too big for my screen size(1366*786). I quickly got to work and scaled down the elements and adjusted the qml code to add some of my customizations.
I managed to achieve the following:
<figure class="video_container">
  <iframe src="https://www.youtube.com/embed/M-p7cHx4OM0" frameborder="0" allowfullscreen="true"> </iframe>
</figure>

The code can be found on [my github repo](https://github.com/lll2yu/sddm-lain-wired-theme).

You can install and try it yourself easily. I have made an [AUR package](https://aur.archlinux.org/packages/sddm-lain-wired-theme) for Archlinux users, you can install it with `makepkg` or using an AUR helper.

Using `makepkg`:

```shell
git clone https://aur.archlinux.org/sddm-lain-wired-theme.git
cd sddm-lain-wired-theme
makepkg -Ccsi
```
#### Install the optional dependencies if  you want sound.

Installation is pretty straight forward for other GNU/Linux distros too
- Make sure you have sddm installed and configured as your default login manager.
- Install the dependencies ```qt5-multimedia``` & ```qt5-quickcontrols``` using your package manager.
- Wav audio codecs are needed to play the sound, make sure you have package(s) that fulfill that installed. 
- Download the [latest release](https://github.com/lll2yu/sddm-lain-wired-theme/releases/latest)
- Decompress the `*.zip` or `*.tar.gz` file 
- Copy all files and directories into `/usr/share/sddm/themes/sddm-lain-wired-theme/` or open a terminal and run `sudo cp -r sddm-lain-wired-theme /usr/share/sddm/themes`

### Setting as default theme
First make sure you have sddm installed and configured as your default display manager

If using systemd
```
sudo systemctl enable sddm.service
```
Make sure other display managers like lightdm,gdm,etc are disabled
```
sudo systemctl disable lightdm.service
```
Now,
- Open up your configuration file `/etc/sddm.conf` and set `sddm-lain-wired-theme` as your current theme

```shell
[Theme]
# Current theme name
Current=sddm-lain-wired-theme
```
- Now you are all set. Enjoy the aesthetic.

Open an [issue](https://github.com/lll2yu/sddm-lain-wired-theme/issues) on github if you face any problem.
