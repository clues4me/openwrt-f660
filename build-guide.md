![OpenWrt logo](include/logo.png)

# Build system usage

    Do everything as an unprivileged user, not root, without sudo.
    Make sure there are no spaces in the full path to the build directory.

    There is an issue affecting the current OpenWrt source tree (from at least 21.02 onwards): OpenWrt images built in certain setups will succeed, but they will hang on boot if installed on a device. To work around this issue, please follow the instructions posted here in the section titled “workaround” before checking out the source tree.

After installing your build system's prerequisites, these are typical build steps:

    # Download and update the sources
    git clone https://git.openwrt.org/openwrt/openwrt.git
    cd openwrt
    git pull
    
    # Select a specific code revision
    git branch -a
    git tag
    git checkout v23.05.0
    
    # Update the feeds
    ./scripts/feeds update -a
    ./scripts/feeds install -a
    
    # Configure the firmware image
    make menuconfig
    
    # Optional: configure the kernel (usually not required)
    # Don't, unless have a strong reason to
    make -j$(nproc) kernel_menuconfig
    
    # Build the firmware image
    make -j$(nproc) defconfig download clean world

The build steps are explained in detail below.
Downloading sources

Clone the Git repository using the following command.

git clone https://git.openwrt.org/openwrt/openwrt.git [<buildroot>]

Possible issues:

    -bash: git: command not found - verify your build system setup.
    fatal: destination path 'openwrt' already exists and is not an empty directory. - remove/rename the buildroot directory.

Updating sources

:!: Sources in development branch change frequently. It is recommended that you work with the latest sources.

git pull

Possible issues:

    -bash: cd: openwrt: No such file or directory - verify the buildroot directory.
    fatal: not a git repository (or any of the parent directories): .git - change the working directory to the cloned buildroot.

Updating feeds

Pull the latest updates for the feeds in case it became outdated.

./scripts/feeds update -a

Make the downloaded package/packages available in make menuconfig.

./scripts/feeds install <package_name>
./scripts/feeds install -a

Possible issues:

    -bash: ./scripts/feeds: No such file or directory - change the working directory to the cloned buildroot.

    Can't exec "make": No such file or directory at ./scripts/feeds line 22.
    Unsupported version of make found: make
    Checking 'gcc'... failed.
    Checking 'working-gcc'... failed.
    Checking 'g++'... failed.
    Checking 'working-g++'... failed.
    Checking 'ncurses'... failed.
    Checking 'awk'... failed.
    Checking 'unzip'... failed.
    Checking 'bzip2'... failed.
    Checking 'wget'... failed.
    Checking 'python'... failed.
    Checking 'python3'... failed.
    Checking 'file'... failed.

    - verify your build system setup.

Creating a local feed

    Prepare your <buildroot> with git cloning openwrt sources from github (e.g. from your fork).
    Create a dir: mkdir -p <buildroot>/my_packages/<section>/<category>/<package_name>.
    Replace the <package_name> with the name of your package.
    e.g. mkdir -p my_packages/net/network/rpcbind.
    The section and category can be found in the Makefile.
    Write a Makefile or download one Makefile from another package, look at samples on github.
    Edit your Makefile and add necessary files, sources...
    More: Creating packages & Creating a package from your application
    Append a line with your custom feed to feeds.conf.default:
    src-link my_packages <buildroot>/my_packages
    Replace the <buildroot> with cloned openwrt sources directory e.g. ~/openwrt
    Move the line with your custom feed above standard feeds to override them.
    Now run: ./scripts/feeds update -a; ./scripts/feeds install <package_name>
    If you are doing this to resolve a dependency you can run ./scripts/feeds install <package_name> one more time and you should notice the dependency has been resolved.
    Build your package.
    Select it in the menu of Make menuconfig
    Build it with make package/my_package/{clean,compile}
    More: Building a single package

Selecting code revision

This step can be skipped you want to build bleeding edge images.
Selecting branch

Each branch contains the baseline code for the release version, e.g. openwrt-18.06, openwrt-19.07, openwrt-21.02, master, and the individual releases, e.g. v18.06.2, v19.07.3, v21.02.0. Each branch is intended to contain stable code with carefully selected fixes and updates backported from the development branch.

To use a branch, clone the Git repository using the git clone command explained above and then move to the branch by using the git checkout command.

# List branches
git branch -a
 
# Use OpenWrt master branch
git checkout master
 
# Use OpenWrt 21.02 branch
git checkout openwrt-21.02

:!: When changing branches, it is recommended to perform a thorough scrub of your source tree by using the make distclean command. This ensures that your source tree does not contain any build artifacts or configuration files from previous build runs.
Selecting tag

Select an individual release tag to install packages from the official release repositories for a long time.

# Fetch and list tags
git fetch -t
git tag
 
# Use OpenWrt 21.02.1 release
git checkout v21.02.1

Selecting hash

Select a specific code revision hash and sync all feeds to the same date.

REV_HASH="4c73c34ec4215deb690bf03faea2a0fe725476f0"
git checkout ${REV_HASH}
REV_BRANCH="$(git rev-parse --abbrev-ref HEAD)"
 
# Replace all src-git with src-git-full: https://openwrt.org/docs/guide-developer/feeds#feed_configuration
sed -e "/^src-git\S*/s//src-git-full/" feeds.conf.default > feeds.conf
 
./scripts/feeds update -a
 
# Edit every line of feeds.conf in a loop to set the chosen revision hash
sed -n -e "/^src-git\S*\s/{s///;s/\s.*$//p}" feeds.conf \
| while read -r FEED_ID
do
REV_DATE="$(git log -1 --format=%cd --date=iso8601-strict)"
REV_HASH="$(git -C feeds/${FEED_ID} rev-list -n 1 --before=${REV_DATE} ${REV_BRANCH})"
sed -i -e "/\s${FEED_ID}\s.*\.git$/s/$/^${REV_HASH}/" feeds.conf
done
 
./scripts/feeds update -a
./scripts/feeds install -a

Using official build config

Compile OpenWrt in a way that it gets the same packages as the default official image:

For example if you are flashing for the archer_c7 you can locate on its wiki page the factory firmware URL https://downloads.openwrt.org/releases/21.02.1/targets/ath79/generic/openwrt-21.02.1-ath79-generic-tplink_archer-c7-v2-squashfs-factory.bin

'ath79' tells you the Target, 'generic' tells you the Subtarget while 'archer-c7-v2' tells you the Target profile for the very specific device.

Download the config by removing everything up to the subtarget 'generic' and adding 'config.buildinfo':

# OpenWrt 19.07 and later
wget https://downloads.openwrt.org/releases/21.02.1/targets/ath79/generic/config.buildinfo -O .config
 
# OpenWrt 18.06 and before
wget https://downloads.openwrt.org/releases/18.06.0/targets/ramips/mt7621/config.seed -O .config

When using this configuration the correct defaults will be already selected for the Target and Subtarget but not for the Target profile so you will have to tailor it for the specific device if you want to build only that one.
Image configuration
Menuconfig

The build system configuration interface handles the selection of the target platform, packages to be compiled, packages to be included in the firmware file, some kernel options, etc.

Start the build system configuration interface by writing the following command:

make menuconfig

This will update the dependencies of your existing configuration automatically, and you can now proceed to build your updated images.

You will see a list of options. This list is really the top of a tree. You can select a list item, and descend into its tree.

To search for the package or feature in the tree, you can type the “/” key, and search for a string. This will give you its locations within the tree.

For most packages and features, you have three options: y, m, n which are represented as follows:

    pressing y sets the <*> built-in label
    This package will be compiled and included in the firmware image file.
    pressing m sets the <M> package label
    This package will be compiled, but not included in the firmware image file, e.g. to be installed with opkg after flashing the firmware image file to the device.
    pressing n sets the < > excluded label
    The source code will not be processed.

When you save your configuration, the file <buildroot>/.config will be created according to your configuration.

When you open menuconfig you will need to set the build settings in this order (also shown in this order in menuconfig's interface):

    Target system (general category of similar devices)
    Subtarget (subcategory of Target system, grouping similar devices)
    Target profile (each specific device)
    Package selection
    Build system settings
    Kernel modules

Select your device's Target system first, then select the right Subtarget, then you can find your device in the Target profile's list of supported platforms.

E.g. to select and save the target for TL-WR841N v11 Wi-Fi router:

    Target System → Select → Atheros AR7xxx/AR9xxx → Select
    Subtarget → Select → Devices with small flash → Select
    Target Profile → Select → TP-LINK TL-WR841N/ND v11 → Select
    Exit → Yes

Configure using config diff file

Beside make menuconfig another way to configure is using a configuration diff file. This file includes only the changes compared to the default configuration. A benefit is that this file can be version-controlled in your downstream project. It's also less affected by upstream updates, because it only contains the changes.
Creating diff file

Save the build config changes.

# Write the changes to diffconfig
./scripts/diffconfig.sh > diffconfig

The firmware make process automatically creates the configuration diff file config.buildinfo, previously named as config.seed in 18.06 and before.
Using diff file

These changes can form the basis of a config file <buildroot>/.config. By running make defconfig these changes will be expanded into a full config.

# Write changes to .config
cp diffconfig .config
 
# Expand to full config
make defconfig

These changes can also be added to the bottom of the config file (<buildroot>/.config), by running make defconfig these changes will override the existing configuration.

# Append changes to bottom of .config
cat diffconfig >> .config
 
# Apply changes
make defconfig

Custom files

In case you want to include some custom configuration files, the correct place to put them is:

    <buildroot>/files/

For example, let's say that you want an image with a custom /etc/config/firewall or a custom etc/sysctl.conf, then create this files as:

    <buildroot>/files/etc/config/firewall
    <buildroot>/files/etc/sysctl.conf

E.g. if your <buildroot> is ~/source and you want some files to be copied into firmware image's /etc/config directory, the correct place to put them is ~/source/files/etc/config.

It is strongly recommended to use uci-defaults to incrementally integrate only the required customization. This helps minimize conflicts with auto-generated settings which can change between versions.
Defconfig

make defconfig

will produce a default configuration of the target device and build system, including a check of dependencies and prerequisites for the build environment.

Defconfig will also remove outdated items from .config, e.g. references to non-existing packages or config options.

It also checks the dependencies and will add possibly missing necessary dependencies. This can be used to “expand” a short .config recipe (like diffconfig output, possible even pruned further) to a full .config that the make process accepts.
Kernel configuration (optional)

Note that make kernel_menuconfig modifies the Kernel configuration templates of the build tree and clearing the build_dir will not revert them. :!: Also you won't be able to install kernel packages from the official repositories when you make changes here.

While you won't typically need to do this, you can do it:

make kernel_menuconfig CONFIG_TARGET=subtarget

CONFIG_TARGET allows you to select which config you want to edit, possible options: target, subtarget, env.

The changes can be reviewed and reverted with:

git diff target/linux/
git checkout target/linux/

Source mirrors

The 'Build system settings' include some efficient options for changing package locations which makes it easy to handle a local package set:

    Local mirror for source packages
    Download folder

In the case of the first option, you simply enter a full URL to the HTTP or FTP server on which the package sources are hosted. Download folder would in the same way be the path to a local folder on the build system (or network). If you have a web/ftp-server hosting the tarballs, the build system will try this one before trying to download from the location(s) mentioned in the Makefiles. Similar if a local 'download folder', residing on the build system, has been specified.

The 'Kernel modules' option is required if you need specific (non-standard) drivers and so forth – this would typically be things like modules for USB or particular network interface drivers etc.
Download sources and multi core compile

Before running final make it is best to issue make download command first, this step will pre-fetch all source code for all dependencies, this enables you compile with more CPU cores, e.g. make -j10, for 4 core, 8 thread CPU works great.

If you try compiling OpenWrt on multiple cores and don't download all source files for all dependency packages it is very likely that your build will fail.

make download

Building images

Everything is now ready for building the image(s), which is done with one single command:

make

This should compile toolchain, cross-compile sources, package packages, and generate an image ready to be flashed.
Make tips

See also: Compiler optimization tweaks

make download will pre-download all source code for all dependencies, this will enable multi core compilation to succeed, without it is is very likely to fail. make -jN will speed up compilation by using up to N cores or hardware threads to speed up compilation, make -j9 fully uses 8 cores or hardware threads.

Example of pre-downloading and building the images on a 4 core CPU:

make -j5 download world

You can use ''nproc'' command to get available CPU count:

make -j $(nproc) download world

or a better macro with nproc+1:

make -j $(($(nproc)+1))

Building in the background

If you intend to use your system while building, you can have the build process use only idle I/O and CPU capacity like this (4 core, 8 thread CPU):

make download
ionice -c 3 chrt --idle 0 nice -n19 make -j9

Building single packages

When developing or packaging software, it is convenient to be able to build only the package in question, e.g. with package jsonpath:

make package/utils/jsonpath/compile V=s

For a rebuild:

make package/utils/jsonpath/{clean,compile} V=s

It doesn't matter what feed the package is located in, this same syntax works for any installed package.

Note: you must have done a full tree build (make, or make world) beforehand for this to work reliably.
Spotting build errors

If for some reason the build fails, the easiest way to spot the error is to do:

make V=s 2>&1 | tee build.log | grep -i -E "^make.*(error|[12345]...Entering dir)"
 
make V=s 2>&1 | tee build.log | grep -i '[^_-"a-z]error[^_-.a-z]' 
(may not work)

:!: If grep throws an error, use fgrep instead.

The above saves a full verbose copy of the build output (with stdout piped to stderr) in ~/source/build.log and shows errors on the screen (along with a few spurious instances of 'error').

Another example:

ionice -c 3 nice -n 20 make -j 2 V=s CONFIG_DEBUG_SECTION_MISMATCH=y 2>&1 | tee build.log

The above saves a full verbose copy of the build output (with stdout piped to stderr) in build.log while building using only background resources on a dual core CPU.

Yet another way to focus on the problem without having to wade through tons of output from Make as described above is to check the corresponding log in logs folder. i.e. if the build fails at make[3] -C package/kernel/mac80211 compile, then you can go to <buildroot>/logs/package/kernel/mac80211 and view the compile.txt found there.
Getting beep notification

Depending on your CPU, the process will take a while, or while longer. If you want an acoustic notification, you could use this way:

make ...; echo -e '\a'

Ignore build errors

If you are building everything (not just the packages to make a flashable image), you will probably want to keep building all packages even if some have compile errors and won't be built.

# Ignore compilation errors
IGNORE_ERRORS=1 make ...
 
# Ignore all errors including firmware assembly stage
make -i ...

Make a summary information of generated image

make json_overview_image_info

Generate a summary of the image (including default packages, type of target, etc... ) in JSON format. The output is available in <BUILD_DIR>/profiles.json.
Calculate checksum for generated files

make checksum

The following action will take place: a checksum will be computed and saved for the output files. This checksum will then be stored in the '<BIN_DIR>/sha256sums' .
Locating images

After a successful build, the freshly built image(s) can be found below the newly created <buildroot>/bin directory. The compiled files are additionally classified by the target platform and subtarget, so e.g. a generic firmware built for an ar71xx device will be located in <buildroot>/bin/targets/ar71xx/generic directory (and the package files are below <buildroot>/bin/packages/mips_24kc).

E.g. if your <buildroot> is ~/source, the binaries are in ~/source/bin/targets/ar71xx/generic and ~/source/bin/packages/mips_24kc.

See also: Directory structure
Cleaning up

You might need to clean your build environment every now and then.

The build artefacts, toolchain, build tools and downloaded feeds & sources files can be cleaned selectively.
The following make-targets are useful for that job.
make clean is the most frequently needed clean operation.
> Cleaned components >
v make argument v	Compiled binaries:
firmware, kernel, packages 	Toolchain
(target-specific) 	Build tools,
tmp/
	Compiled
config tools 	.config

	feeds, .ccache,
downloaded source files
clean 	x 					
targetclean 	x 	x 				
dirclean 	x 	x 	x 	x 		
config-clean 				x 		
distclean 	x 	x 	x 	x 	x 	x
Clean

make clean

Deletes contents of the directories /bin and /build_dir. This doesn't remove the toolchain, and it also avoids cleaning architectures/targets other than the one you have selected in your .config. It is a good practice to do make clean before a build to ensure that no outdated artefacts have been left from the previous builds. That may not be necessary always, but as a general rule it helps to ensure quality builds.
Targetclean

make targetclean

This cleans also the target-specific toolchain in addition of doing make clean. This may be needed when the toolchain components like musl or gcc change.
Does a make clean and deletes also the directories /build_dir/toolchain* and /staging_dir/toolchain* (= the cross-compile tools).

Note: make targetclean has been introduced in 22.03 and is not found in earlier OpenWrt versions.
Dirclean

make dirclean

This is your basic “full clean” operation. Cleans all compiled binaries, tools, toolchain, tmp/ etc.
Deletes contents of the directories /bin and /build_dir and /staging_dir (= tools and the cross-compile toolchain), /tmp (e.g data about packages) and /logs.
Distclean

make distclean

Nukes everything you have compiled or configured and also deletes all downloaded feeds contents and package sources. :!: In addition to all else, this will erase your build configuration <buildroot>/.config. Use only if you need a “factory reset” of the build system!
Selective cleanup

In more time, you may not want to clean so many objects, then you can use some of the commands below to do it.

# Clean linux objects
make target/linux/clean
 
# Clean package base-files objects
make package/base-files/clean
 
# Clean luci objects
make package/luci/clean

Examples

    https://github.com/mwarning/openwrt-examples
    https://forum.openwrt.org/viewtopic.php?pid=129319#p129319
    https://forum.openwrt.org/viewtopic.php?id=28267

Troubleshooting

    Beware of unusual environment variables.
    First get more information on the problem using the make option make V=sc or enable logging.
    Read more about make options: Buildroot Techref.

Missing source code file, due to download problems

First check if the URL path in the make file contains a trailing slash, then if it does, try with it removed (helped several times). Otherwise try to download the source code manually and put it into dl directory.
Compilation errors

Try to update the main source and all the feeds, however beware of other potential problems. Check for related issues in the bugtracker, otherwise report the problem there mentioning the package, target (CPU, image, etc.) and code revisions (main & package).

Some packages may not be updated properly and built after they got stuck with old dependencies, resulting in warnings at the beginning of the compilation looking similar to:

WARNING: Makefile 'package/feeds/packages/openssh/Makefile' has a dependency on 'libfido2', which does not exist

The build environment can be recovered by uninstalling and reinstalling the failing package

$ ./scripts/feeds uninstall openssh
Uninstalling package 'openssh'
$ ./scripts/feeds install openssh
Installing package 'openssh' from packages
Installing package 'libfido2' from packages
Installing package 'libcbor' from packages

WARNING: skipping <package> -- package not selected

Run make menuconfig and enable compilation for your package. It should be labeled with <*> or <M> to work correctly.
Flashable images for my device are not generated

When you execute make to build a flashable image for your device, both a sysupgrade and a factory image should be generated for every board that is linked to the device profile that you have selected via make config or make menuconfig.

If running make does not yield images for one (or even all) of the boards linked to the device profile that you have selected, than you probably have selected/enabled too many options or packages, and the image was too big to be flashed onto your device.
