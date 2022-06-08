---
layout: post
title: Trying NixOS on the desktop
---

A few weeks ago, I decided to install [NixOS](https://nixos.org/) on my laptop. I'm barely using it these days, and I already had Pop_OS installed on my work computer, so I could experiment with alternate distributions without too much risk.

NixOS is a Linux distribution built around the Nix package manager. It's a distribution-wide generalization of the "bundle your dependencies" approach that [has packagers tearing their hair out](https://blogs.gentoo.org/mgorny/2021/02/19/the-modern-packagers-security-nightmare/) these days.

The core idea is that, much like a Docker container or a Rust program, a Nix package is bundled with an unique version of its dependencies (instead of sharing them will all other dependencies on the system). This trivially avoids dependency problems where one application needs `mygraphixlib <= 1.3` and another needs `mygraphixlib >= 1.5`, and you can't install them both at the same time because they both expect to read the same `/usr/lib/mygraphixlib.so.1` file.

The frequently cited downsides are that packages are heavier (because they don't share dependencies) and security updates are harder to roll out, because you need to update every package instead of updating a single `.so` library. I don't know how relevant either of these claims are in practice. (NixOS didn't seem too heavy when I installed it)

Now, NixOS is a mostly server-centric distrib. The site's tagline is "Reproducible builds and deployments": it markets itself as a way to ship containers, build reliable CIs, easily configure servers, etc.

So, I was curious how good the desktop experience was for NixOS. In particular, I was looking for small workflow interruptions; the things that would sneak up on you and completely halt the work you're trying to do until you resolve them. This is how a desktop distribution should be judged: by how much it gets out of your way, when you want to do things quickly.

(Spoiler: some things went better than I expected, but overall I didn't really like the experience. I'll stick to Pop_OS.)


## Installation

I'm following the instructions of the [Manual page](https://nixos.org/manual/nixos/stable/index.html).

First I write a NixOS image to a USB stick, and then I boot from it.

### Booting

First surprise: NixOS doesn't have an installation assistant. It expects you to use `parted` from the command line to partition your disk, format the partitions, etc.

The instructions are fairly detailed, eg:

> Add the root partition. This will fill the disk except for the end part, where the swap will live, and the space left in front (512MiB) which will be used by the boot partition.
>
> ```sh
> parted /dev/sda -- mkpart primary 512MiB -8GiB
> ```

5 minutes in, NixOS is already making a statement: it is a distribution for power users. Non-technical users who don't know how to use the Linux command line simply aren't the target demographic. Still, I'm able to follow the instructions easily enough (and in the process, got a better understanding of how partitions work 🎉).

One genuine criticism I have: the instructions explain *what to do*, not *what the system should look like afterwards*. This is bad, because if the user makes a mistake along the way, they won't know where it is and how to fix it, and will have to start over from the beginning.

The manual should include one or two calls to `parted /dev/sda -- print`, both to teach the user that they can use it to see what they're doing, and to reassure them they're on the right path.

In general, any tutorial about installing a complex system should be careful to degrade gracefully. Getting a single step wrong shouldn't doom the user to throw away all the progress they've made.

### Config

NixOS revolves around a `configuration.nix` file.

When installing it on a new machine the first thing you do after formatting the drives is to write that file, and generate an installation from it.

Anyway, after looking at various examples, the documentation for various options, etc, I write [my config file](https://gist.github.com/PoignardAzur/5f8d90c0be4640797efc66323f26bb2a).

I run `nixos-install`, which downloads and installs the packages I listed, and prompts me for the root password. Then I reboot.

### First login

Once the installation is over, I start NixOS for the first time. There are still some hurdles to pass.

The user account I've doesn't show up in the login display manager. This is because I need to log in as root first, and use `passwd` to set a password for your user account.

This seems like accidental complexity. The installer should just ask you to set your main account's password at the same time it asks you to set your root password (in a desktop computer, they're probably the same).

In fact, you could generalize that to other credentials: Google accounts, Firefox sync, Github, Gitlab, etc. These are all credentials you probably don't want to be storing in plaintext in your `configuration.nix` file, but you can probably enter them at the same time you're installing your packages. You could even provide a password database from your password manager (KeePass, LastPass, etc) to automate the process.

I'm not just speculating out loud: this would legitimately improve the NixOS UX. Speaking as someone who has switched through a fair amount of distributions, one of the things that annoys me the most is having to re-enter credentials and write config files all over again. Firefox Sync and Nextcloud help, but the process could be streamlined.

### Welcome screenshot

When I first log in, you get the standard "Welcome to GNOME" window.

This window should probably be supressed. It asks you to set parameters that have already been set in your config file.

Also, it kind of hurts the brand image of Nix to have a generic Debian welcome screen. Nix should probably start with a personalized "Welcome to NixOS" message.

(Also also, I think the screen shows up every time you rebuild your OS with `nix-rebuild`, which is a little silly.)


## User experience

### Desktop environment

First major disappointment: there is no Pop Shell package.

This is a bummer. [Pop_OS](https://pop.system76.com/) (no, I'm not spelling it with a `!`) is a game-changer for desktop UI: it has great window tiling, great keybindings, and good defaults for about everything. Pop_OS COSMIC is the first desktop environment I used that I liked better KDE Plasma.

So it's a shame that there's no official NixOS package for it. There's [some discussion of the issue](https://github.com/NixOS/nixpkgs/issues/92769) where people have written "homemade" configs, but I haven't been able to make them work.

### Terminal

The desktop doesn't bind `Super + T` to "open a terminal" by default, something most distributions I've tried do.

Also, searching "term" in the Applications grid offers both the GNOME Terminal and Konsole, which is a little odd. I'm pretty sure I didn't include either of them in my `configuration.nix` *or* install them manually, so I'm guessing Nix installs both by default. Again, a little strange.

NixOS has a setting to use fish as your default shell, and it's mostly painless.

One problem: when using an unknown command, fish soft-crashes with this error message:

```
DBI connect('dbname=/nix/var/nix/profiles/per-user/root/channels/nixos/programs.sqlite','',...) failed: unable to open database file at /run/current-system/sw/bin/command-not-found line 13.
cannot open database `/nix/var/nix/profiles/per-user/root/channels/nixos/programs.sqlite' at /run/current-system/sw/bin/command-not-found line 13.
```

Both bash and sh give the correct `foobar: command not found` error message, so I'm guessing the problem is with the fish integration.


### Package management

This part is where we *really* see the signs that NixOS is a distribution for container deployment first and foremost. NixOS is in *dire* need of a better package management experience.

The blessed way to install a package is to edit the `configuration.nix` file, and rebuild the system. It's fast, it's efficient, it can be done with no downtime, it's error-resistant, it's great for hosting containers, and *it's completely inappropriate for a desktop OS*.

The recommended quick-and dirty way to install a new package from within a session is essentially to run `nix-env --install -A nixos.MY_PACKAGE_NAME`. I have several problems with that feature:

- The syntax is a bit obscure. It's not immediately clear what the `-A` stands for in the above expression. It's even less clear in the more commonly cited `-iA` format.
- It mutates hidden state in a way that clashes with the declarative nature of Nix. I wish it were more like `cargo-add`, and would programmatically add lines to a config file, while using that file as a source of truth. (Technically, you can do that manually with `nix-env --query`.)
- The error messages are bad. When I enter `nix-env -iA clang`, the error I get is:

    ```
    error: attribute `clang` in selection path `clang` not found
    ```

    If you don't already understand how `.nix` config files work, you have *no idea* what that means. Whereas I'd want something like:

    ```
    error: cannot find package `clang` - did you mean `nixos.clang`?
    ```

The second most important part of the experience is browsing packages. Ideally, I want a way to browse with a GUI app like Pop Shop or a terminal utility like `yaourt` and `yay`. The point is, I want something where I can enter an app's name (eg `ripgrep`) or a keyword list (eg `password manager`) and quickly know whether the distribution has the package I'm searching for, and some quick information about it. In that regard, NixOS is middle-of-the-road.

The `yay` equivalent in Nix is `nix search`. I think it could be advertised better, because I completely forgot about it for a while. The format is okay, and it helps you quickly figure out whether a given package exists.

NixOS also has [search.nixos.org](https://search.nixos.org), which performs the role of an app browser. It's not as integrated as an app, and it includes "infrastructure packages" that can clutter searches, but it's serviceable. It's a little bare-bones: no screenshots, no logos, etc. More importantly, it has no curation that I can see; there are no recommended apps, no user ratings, etc.

(Note that almost everything I criticized here also applies to Debian/Ubuntu. The browsing experience of `apt` is kind of terrible.)

One thing that Ubuntu has by default and NixOS lacks: in Ubuntu, when you enter a command that you don't have installed, the shell prints a list of packages you could try to install to get the command. I wish Nix had something similar.

I'm mostly nitpicking. Installing a package on NixOS is easy 95% of the time. But the whole thing has a "made by sysadmins for sysadmins" aesthetic that can be off-putting for newcomers.


### Applications

I try to install the following apps:

- Firefox
- Chromium
- Kate
- Intellij Idea
- Atom Editor
- Mattermost
- Alacritty
- Gimp
- VLC
- Blender
- qBitTorrent
- Nextcloud-client
- LibreOffice
- OnlyOffice

OnlyOffice is the only one of these that doesn't have a package. Most installations were painless.

#### Nextcloud

I was confused at first because NixOS offers a lot of Nextcloud versions, but those are actually Nextcloud servers. I've installed the `nextcloud-client` package, but it doesn't appear in the Applications menu, and it doesn't seem to be running in the background on startup.

(Also, is it just me or is the new Nextcloud interface terrible? The old interface, which has the infos and settings I want, is hidden behind a submenu, and the new interface is just a bunch of HTTP links that open in the browser. This feels like a metrics-driven corporate rebranding that decided web pages were the hot new thing and regular menus were boring. The thing is, if I want to access the Nextcloud site, I can already open my browser by myself. Making my local synchronisations harder to manage doesn't help me in any way.)

I *think* the Nextcloud integration would have worked better if I'd installed it with Home Manager? I don't know.

#### Wallpaper

Searching "Wallpaper" in search.nixos.org gives me 29 packages, starting with:

- haskellPackages.wallpaper
- haskellPackages.xmonad-wallpaper
- gnomeExtensions.random-wallpaper
- xwallpaper
- gnomeExtensions.google-earth-wallpaper

Three of these fail to install, none of them do what I want.

Among other results, Nitrogen changes the wallpaper, but crashes when doing so. Fondo works when I install it, but is only available as com.github.calo001.fondo in the command line (presumably because it's a Flatpak app?).

I can't find a HydraPaper package; it's too bad, because the other apps I've mentioned don't have as many features.

#### Clipboard management

I try to find a clipboard history widget.

- There is no Diodon package.
- I manage to install *CopyQ* and *Clipboard Indicator*, but neither of them seem to do anything.


### Video games

I try to install three games:

- Battle for Wesnoth
- Minetest
- Teeworld

(Honestly, *Battle for Wesnoth* is the only quality open-source game on Linux that I know of; open-source games are garbage. Nothing personal, open-source developers, gamedev is just insanely hard.)

All three games install without a problem.

I install Steam, and get the following error on startup:

```
You are missing the following 32-bit libraries, and Steam may not run: libpipewire-0.3.so.0
```

Well, that was fast. I'm not playing games on NixOS any time soon.


### Home Manager

In this section so far, there's one tool I've barely mentioned: Home Manager. I get the feeling some readers are mad at me, and ready to scream to me "Of course you have problem X, you didn't use Home Manager"!

The thing is, Home Manager confuses me. It's a non-official community-favorite tool, meaning it's not installed by default, it's not recommended in the manual or any of the resources on the NixOS website, but the reddit and discourse threads all assume you've installed it.

It was unclear to me at first what HM was for. The README says:

> This project provides a basic system for managing a user environment using the Nix package manager together with the Nix libraries found in Nixpkgs. It allows declarative configuration of user specific (non global) packages and dotfiles.

Okay... so it's for installing packages locally? But you already have `nix-env -i` for that? And if you're going to write declarative package lists, why not just edit your `configuration.nix` file? And if you're going to synchronize your dotfiles, why not just put your `.config/` folder in Nextcloud?

Also, the very next section of the README says:

> Unfortunately, it is quite possible to get difficult to understand errors when working with Home Manager, such as infinite loops with no clear source reference. You should therefore be comfortable using the Nix language and the various tools in the Nix ecosystem.

"Please make sure you've spent at least two weeks installing servers with Nix before trying to use Nix on your laptop. Remember that Nix is meant for hosting containers, and that "personal computer" thing you're doing with it is purely a side project of ours." That's encouraging!

Anyway, I'm realizing now that Home Manager is meant to help you with what I was gesturing towards in **First login**: giving you a single source of truth that your entire system is installed from, including user apps and config files.

I'm just not sure the format is great for that:

- First, the config file should be in JSON or TOML or some other very basic file format. Extensibility is good, but I shouldn't have to learn a package manager's turing-complete language just to synchronize my dotfiles.
- Second, the config file should be machine-extensible. Again, like with `cargo-add`, you should be able to say "I want to install a new app" and have your config file automatically changed for you.
- Three, attributes like `programs.git` that generate a dotfile should be avoided. I already have a dotfile. I'm using it and updating it on other machines. I want to be able to reuse that file as-is, not have to rewrite it into Nix's special syntax.

Point is, I wasn't comfortable with Home Manager, and that's why I barely mentioned it in this section.

Anyway, that's my experience installing and using apps on NixOS. Next comes the developer experience.


## Developer experience

To be clear, I'm writing this as a non-sysadmin developer. I've almost never installed servers or hypervisors in my life, and I don't intend to start developing Nix packages. This is my experience writing regular code with Nix.

### Using Rust

I start with Rust, since it's the language I use the most these days.

In the middle of writing my first command, I get a `no override and no default toolchain set` error message (because fish autocomplete tries to call a rust toolchain to complete my command and can't find one). This one might be my fault.

After installing various toolchains, I try to compile a few packages:

- **ripgrep:** No problem. Compilation goes without a hitch, tests all pass immediately, I can use the resulting binary to grep things. Great.
- **tokio:** Compilation has a few minor errors, I quickly fix those, and tests pass.
- **druid:** Compilation immediately fails when building the `glib-sys` crate. This is because the crate requires `pkg-config` to build, which means it basically requires global state.

So, the results aren't too surprising there. A portable CLI utility that takes text and outputs text compiles without a hitch. A semi-portable server framework with some target-specific flags can be compiled with fixups. A GUI framework that relies on globally-installed foundational libraries fails to compile.

I'm curious how well those results would replicate across different ecosystems. Rust's GUI ecosystem is notoriously immature (unlike, say, its CLI ecosystem). Hopefully the ecosystem will eventually coalesce around a few foundational crates that handle platforms and asset management, and these crates will also handle compiling on Nix.


### The Bash problem

A NixOS environment can make it a little harder than expected to build projects with install scripts.

This is because a lot of scripts will start with this line:

```bash
#!/bin/bash
```

The problem is, NixOS doesn't have a file in `/bin/bash`. It has a `/bin/sh` and a `/usr/bin/env` for POSIX compliance, and that's it. It *does* have at least one bash binary; it's just hidden somewhere in `/nix/store/`.

The way I understand it, [this is a contentious issue](https://discourse.nixos.org/t/add-bin-bash-to-avoid-unnecessary-pain/). There's an argument to be made that NixOS should simply have a symlink or something in `/bin/bash` to avoid unnecessary compatibility problems.

Other people think that this is a bad idea; that running bash scripts directly is a form of global state that shouldn't be encouraged, and that there is no obviously correct file that should be put in `/bin/bash` (because two different versions might have different behaviour, for instance, which hurts reproducibility).

The encouraged solution is asking maintainers to switch their shell scripts to

```bash
#!/bin/env bash
```

The thing is, I don't think this is a very satisfying solution. It works for packagers who are porting a project to Nix, because even if the maintainer isn't willing, they can always add a small patch to their package description.

But, if you're a random user, this is all very inconvenient. Say you've just cloned a project on Github, and you're trying to get a feel for it. What you really want is an installation process that goes smoothly from the `git clone` to the point you're running the examples. Having to manually go in shell scripts and edit the shebangs is a demoralizing obstacle.

I'm not sure there's a solution, though. I think ideally, what I'd want is an utility that would parse install scripts (and CMakefiles and whatnot) and output a local environment to build the project. I could then use `nix-env` to build the project from inside this environment; maybe it would even pretend to be a Debian or an ArchLinux for maximum compatibility.

The goal wouldn't be to create a reproducible package, just to generate an environment on a best-effort basis that I can quickly start hacking in.

But as it is, you just have to hope the projects you're working on use standardized build processes with no shell scripts or extra dependencies.


### Compiling VLC

VLC is an interesting stress-test for a distribution. The app itself is extremely portable, but the compilation process is complex enough that the team distributes containers that set up the entire environment the project can be compiled in.

So, I clone the VLC repo, and set out to build it.

#### Autoconf

First I run the `./bootstrap` script. It emits an error that goes `autoreconf: command not found`. After a quick search, I end up installing `autoconf` and `automake` and other tools. I restart the script over and over and install the things it complains about missing.

Eventually, autoconf gives me a bunch of `error: possibly undefined macro` messages and I give up.


#### Docker

As I said earlier, VLC maintainers provide Docker images to help build their project. These images can be loaded using [the denv utility](https://gitlab.com/garfvl/denv/), with minor caveats.

I clone the repository and create an alias to the denv binary. Then I run denv on the VLC repo and...

```
> denv compile-vlc-stable
[INFO] Exporting current directory (/home/olivier-faure/Documents/vlc) inside the container
[INFO] Detecting VLC_LATEST tag in profile
[INFO] Using Docker image vlc-debian-stable with tag 20190716183804
ERROR: Couldn't connect to Docker daemon at http+docker://localhost - is it running?
```

Oh come on!

So, let's see... I have the docker package installed... maybe this is a problem with the denv binary? I try to run the "hello world" container:

```
> docker run hello-world
docker: Cannot connect to the Docker daemon at unix:///var/run/docker.sock. Is the docker daemon running?.
See 'docker run --help'.
```

Nope, it's a problem with the docker daemon. After a few tries and some search, I find that I need to set `virtualisation.docker.enable = true;` in my nix config. I do so and reboot the system.

```
> docker run hello-world

Hello from Docker!
This message shows that your installation appears to be working correctly.
```

Good, docker is working now. I run `denv` again and...

```
> denv compile-vlc-stable
[INFO] Exporting current directory (/home/olivier-faure/Documents/vlc) inside the container
[INFO] Detecting VLC_LATEST tag in profile
[INFO] Using Docker image vlc-debian-stable with tag 20190716183804
ERROR: Couldn't connect to Docker daemon at http+docker://localhost - is it running?
```

The exact same error. Also, `denv` works on my main machine, so I'm pretty sure the problem is related to NixOS.

Well, that's it. I'm done.

I've already spent way more days on this than I intended to at the onset, and I still don't have an environment as convenient as Pop_OS was after two days of configuration. I think it's time I cut my losses.



## Conclusion

So, do I think NixOS is ready for desktop use?

Eeeh... maybe.

I've been extremely negative, because I think UX is all about small inconveniences. Overall, I don't think NixOS is that bad. If you're already using it to deploy services, you can probably use it for your PC too. Otherwise, I think Pop_OS provides a better experience by default.

With that in mind, how could NixOS be improved for desktop users?

### Support Pop Shell

Seriously.

Having good defaults is *the single most important* quality of a desktop OS. And this is something Pop_OS is very good at. NixOS should have official support for the Pop_OS desktop manager.

### Better Nextcloud integration

Part of the appeal of NixOS is the idea of having a very small source of truth, deploying it on a new computer, and having the new machine immediately feel like "home" with minimal configuration.

I think part of this should be integrating with Nextcloud. You shouldn't even need to log in for the first time. In the ideal process, you'd just add your Nextcloud username to a config file, have the installer prompt you for your Nextcloud password (or a one-time token or whatever), and have the installer download your files and set up your `/home` at the same time it's installing packages.

### Online playground

Many modern programming languages have what's called a "playground": an online platform where you can quickly write code without worrying about installing the toolchain or polluting your `~/Documents` folder.

Nix is modular enough it could probably do the same.

You would give users a space they could paste a configuration file into, and then the playground would generate an entire OS, X server included. That way, neophyte users could experiment and visualize what they're about to install without having to download an ISO or set up a VM.

You could even build a "configure your OS" app that asks:

- Whether you want dark mode,
- Which desktop managers you want,
- Which browsers you want,
- etc,

All with choices presented visually. The config file generated from that app would be loaded in the playground, so the user could preview what their choices look like.

### How much this matters

Does anybody care about this?

That's an open question. [The download page](https://nixos.org/download.html) explicitly says:

> Please note that NixOS at the moment lacks a nice, user-friendly graphical installer. Therefore this form of installation may not be suitable for novice Linux users.

And sure, maybe they mean this temporarily, but the project has existed for 18 years. Maybe they're just comfortable being a distro for power users by now.

On the other hand, this is an open-source project. The community drives the project roadmap. And I know there's a lot of people who are interested in the concept of a declarative OS for the desktop.

So, for anybody who *is*, interested, this is my take. This has been my experience, trying to use NixOS for the desktop. Some of it is surprisingly smooth, some of it is painful. But I think it's not that far away from a point where even moderately tech-savvy users might seriously consider switching to it.

I really hope it gets there.

Discussion on [r/NixOS](https://www.reddit.com/r/NixOS/comments/plq4nl/blog_post_trying_nixos_on_the_desktop/).

*2022-06-08 update: Since writing that blog post, I came across https://ianthehenry.com/posts/how-to-learn-nix/, which is another take on the same idea (a diary of installing Nix, describing its papercuts along the way), except infinitely more detailed. I recommend that post both to anyone wanting to either install Nix and to Nix maintainers hoping to improve the UX.*
