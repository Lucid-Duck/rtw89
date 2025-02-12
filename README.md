rtw89
===========
### Prerequisites

git, make, gcc, kernel-headers and dkms (dkms is needed only if you want to install this driver via [dkms](https://en.wikipedia.org/wiki/Dynamic_Kernel_Module_Support).)

### Usage

```
git clone https://github.com/a5a5aa555oo/rtw89
cd rtw89
make clean modules
sudo make cleanup_target_system (This step is optional.)
sudo make install
sudo make install_fw
```
or
```
git clone https://github.com/a5a5aa555oo/rtw89
cd rtw89
sudo make cleanup_target_system (This step is optional.)
sudo dkms install $PWD
sudo make install_fw
```
If your system has [Larry's rtw89 driver](https://github.com/lwfinger/rtw89) installed, you must run `sudo make cleanup_target_system` to remove that driver, or this driver won't work correctly.

### Note
Kernel 6.6 ~ 6.14 are supported. For kernel 5.15 ~ 6.5 users, use the rtw89 driver in [6.6-lts branch](https://github.com/a5a5aa555oo/rtw89/tree/6.6-lts) instead.

Tested only with RTL8852BE on Arch Linux (kernel version: 6.6.75-1-lts66 / 6.12.13-1-lts) and it works.
