# Arch Linux Kernels / Docs / Headers for GPD Pocket

## Installing Kernels on Arch
To install these kernel packages, all three packages (kernel, docs and headers) must be installed.

    sudo pacman -U linux-*

## Available Kernel Versions

| Version | Wifi | Bluetooth | USB-C Data | Crackle-free Audio | Notes |
| --- | --- | --- | --- | --- | --- |
| [4.14.0-rc1-1](4.14.0-rc1-1) | ✔ | ✔ | ✘ | ✘ | |
| [4.14.0-rc2-1](4.14.0-rc2-1) | ✔ | ✘ | ✔ | ✘ | |
| [4.14.0-rc3-1](4.14.0-rc3-1) | ✔ | ✔ | ✔ | ✘ | |
| [4.14.0-rc3-1-audio](4.14.0-rc3-1-audio) | ✔ | ✔ | ✔ | ✔ | `CONFIG_INTEL_ATOMISP=n` |
| [4.14.0-rc4-1](4.14.0-rc4-1) | ✔ | ✔ | ✔ | ✔ | * [Hans' rt5645 clock audio fix](https://github.com/jwrdegoede/linux-sunxi/commit/16c985c48a36456cef1bd0dc15b8fb5ab4fd7c77)<br />* `CONFIG_INTEL_ATOMISP=n` |


A gpd-fan package is also [available here](gpd-fan).

# Credits
All packages built with [njkli's PKGBUILD](https://github.com/njkli/gpd-pocket/tree/master/linux-jwrdegoede).