# Secure your Debian computer with your GitHub YubiKey

The [GitHub Yubikey](https://www.yubico.com/why-yubico/for-individuals/github/) is a cool little device, but you can use it to secure a lot more than just your GitHub account. Here's how I set up my YubiKey to unlock my Debian computer.


> **Can I follow these instructions if I'm not using Debian?**
Sort of. I'm trying to suss out how to get it working on Fedora, but I'm having trouble with the `check` unit testing framework. It should theoretically work on any GNU/Linux, UNIXy thing, or Mac OS, but your mileage may vary. Feel free to message me on [Twitter](https://twitter.com/brandon_mn) if you have questions -- I'm glad to help!

There are three new pieces of software you'll need to set up in order to get this going -- `libu2f-server`, `libu2f-host`, and `pam-u2f` -- all three maintained by Yubico. You'll also need a 2FA hardware device that supports the [U2F specification](https://fidoalliance.org/specifications/overview/). You can get such a device from [Yubico directly](https://www.yubico.com/products/yubikey-hardware/fido-u2f-security-key/).

Each of them has a rather distinct build process. We'll be compiling and installing software from source, but if you're not familiar with what that means, that's okay. 

> *The only requirement to complete this tutorial is a working knowledge of some sort of Unix command line, and some sort of Unix-like system to run it on.* If that's all gibberish to you, no worries -- I recommend starting with [The Command Line Crash Course](https://www.yubico.com/why-yubico/for-individuals/github/).

The Yubico developer documentation is pretty easy to follow, but I've made some notes below of some details that aren't fully described there. Follow along with my notes, and I think you'll be up and running with your fancy key in no time!

Let's dive in.

### First thing's first: install dependencies
Before we even think about plugging in our U2F key, we need to make sure our computer has the software it needs to work with the keys.

If you're using Debian, you can run the following command to install all of the dependencies at once. 

```
 sudo apt-get install autoconf automake libtool check pkg-config \
 libglib2.0-dev libjson0-dev libssl-dev help2man gtk-doc-tools gengetopt \
 libhidapi-hidraw0 libhidapi-dev libpam-dev asciidoc xsltproc libxml2-utils \
 docbook-xml --no-install-recommends
```

If you're not running Debian, or just want to know what all these applications do, read further -- I've included a ton of info on these dependencies later in the tutorial.

### The base layer: libu2f-server
This is the start of it all: the server talks to your computer's [device event handling system](https://en.wikipedia.org/wiki/Udev), and waits for a U2F device to be plugged in.

> [Read more about how libu2f-server works on the Yubico Developer site](https://developers.yubico.com/libu2f-server/).


DEPENDENCIES:

| name | debian name | fedora name | listed in developer docs? |
| ---  | ---    | ---    | ---     |
| autoconf | autoconf | autoconf | yes| 
| automake | automake | automake | yes |
| libtool | libtool | libtool | yes |
| check | check | check ??? | yes |
| pkg-config | pkg-config | pkg-config | sort-of? |
| glib2 (devel) | libglib2.0-dev | ??? | no (found on compiler warning) |
| json-c (devel) | libjson0-dev | json-c-devel | yes |
| opessl (devel) | libssl-dev | openssl-devel | yes |
| help2man | help2man | help2man | sort-of |
| gtk-doc-tools | gtk-doc-tools | gtk-doc | no (found on compiler warning) |
| gengetopt | gengetopt | gengetopt | no (found on compiler warning) |

Otherwise, follow the steps on the page.

In your shell:
1. `   autoreconf --install `
2. `./configure --enable-gtk-doc`
3. `make check && sudo make install`

If all goes according to plan, cool! You're in good shape. Install the next thing.

## libu2f-host
The host talks to the server, and completes the other half of the authentication call.

> [Read more about how libu2f-host works on the Yubico Developer site](https://developers.yubico.com/libu2f-host/).

In addition to the dependencies listed above, this package requires the following tools to be installed via the package manager of your choice (Debian `apt-get` and Fedora/RPM `yum` names listed below).
| name | debian name | fedora name | listed in developer docs? |
| ---  | ---    | ---    | ---     |
| hidapi | libhidapi-hidraw0 | hidapi | yes |
| hidapi (dev) | libhidapi-dev | hidapi-devel | yes |

Then, build things with the following shell commands: 

1. `make`
2. `./configure --enable-gtk-doc`
3. `make check && sudo make install`

### pam-u2f

In addition to the dependencies listed above, this package requires the following tools to be installed via the package manager of your choice (Debian `apt-get` command listed below) 

`apt-get install autoconf automake libtool libpam-dev asciidoc xsltproc libxml2-utils docbook-xml --no-install-recommends`

Then, run:

1. `autoreconf --install`
2. `./configure`
3. `make check`
4. `make install`

> It's possible the `make install` you ran for the previous two packages put your .so files in /usr/local/lib, which would cause `make check` to fail here. If that's the case, just copy libu2f-* from /usr/local/lib to /lib/, and start from command 3.

### Teach Your Machine to Speak YubiKey

#### `udev` rules

Unless you've explicitly indicated otherwise, Debian's device management system (`udev`) treats your YubiKey just like any other USB drive.
To cajole `udev` into alerting our freshly-instaled software when a YubiKey is plugged in, we must add the following file to your `/etc/udev/rules.d` folder:

https://raw.githubusercontent.com/Yubico/libu2f-host/master/70-u2f.rules

#### User-Yubikey Mappings
In order for the u2f server to log you in, it needs to associate your Yubikey to your user account.
This can be done with a u2f_mappings file, stored someplace like `/etc/u2f_mappings`.

Here's mine:  
`brandon:yubikey-id`

You can get your Yubikey ID with the following command:

```
pamu2fcfg -u $(whoami) -opam://hostname \
-ipam://hostname
```

If you've specified an App ID and Origin ID (-ipam and -opam respectively), use that instead of the hostname. The above command assumes that you are *logged in as yourself (not the superuser)* and *did not set the App or Origin ID to something other than your hostname*.

#### `pam` schemes

Next, you can add `pam_u2f.so` (the library we built above) as an authentication method to any authentication scheme in `/etc/pam.d/`.

You can add U2F authentication to `sudo` quickly and easily, by editing the `sudo` file in `/etc/pam.d`

Add this line wherever you want the authorization to take place. Keeping with our `sudo` example, we'll make it so a YubiKey will log you in *instead* of your password. In other words, we want our YubiKey to be *sufficient* to let us run commands with `sudo`, but not necessary.

To accomplish this, I placed the following line at the very bottom of the `sudo` file.

```
  
  ## Allow YubiKey auth
  auth sufficient pam_u2f.so authfile=/etc/u2f_mappings

```

If you'd like to use your YubiKey as a true second factor (along with password authentication), you can do so by replacing `sufficient` with `required` in the above directive.

Then, reboot your computer to restart `pam`, and you're in good shape! Try to execute a command as `sudo` with your YubiKey plugged in, and it will start blinking.


The fun thing about `pam` is that it controls nearly every place you log in on your computer -- check out what the other files in your `/etc/pam.d` do, and you can add YubiKey 2FA to them as well, if you like!

> **Questions? Corrections? Updates? High-fives?**
Let me know -- open an Issue on GitHub, or [@ me on Twitter](https://twitter.com/brandon_mn)!

-----

#### TODO
* add images
* review for continuity
* remove ambiguity re: dependencies
* test for other OSes?
