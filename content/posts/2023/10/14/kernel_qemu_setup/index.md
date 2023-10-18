---
title: Kernel QEMU setup for development and debugging
tags: ["kernel", "debugging", "qemu", "setup", "linux"]
toc: true
---

## Introduction

Setting up kernel for debugging can be a confusing task for a beginner. [It certainly was for me when I was starting out](/posts/2022/08/26/the_curious_mouse_click).

In this post, I'll outline a way to quickly set up your computer to compile and run kernels on the fly. You need to never hassle about it again, we will make aliases along the way so that you just have to use simple commands to do things. Tab completion auto-filling the command name makes things very convenient. 

Of course, one should know what an alias is doing before adding them for general use in `.bash_aliases`, so please make sure you understand everything and only then add it. Hopefully, I'll be clear, and the context will be obvious if I not mention some things. I assume basic familiarity with the kernel.

For those who have already read the article and just wants to see the bash aliases together, I have them (and a couple more) in the following gist: https://gist.github.com/siddhpant/0af22835e6825b90869f583749af2070

I may update the gist, but not this post necessarily.

---

## Cloning and saving the kernel

If you ever tried to clone the kernel, you'd know the download is huge! The history is long, so it makes sense. But it does not make sense to keep cloning it again and again.

Clone entire history once, then keep updating it (using `git pull`). Whenever you want to work on the kernel, either

-   Use `git worktree`, which copies the tree but shares the `.git` across all of them. Note that it is the `.git` folder which is the biggest (currently it is of 2.9 GB).
-   Copy the entire repo using `cp`.

Do this for **each** bug fix or patch set, i.e., for related work. This will help in multitasking and saving work. If you use copy-on-write filesystems like BTRFS, the copy won't even actually take extra space. Only upon modifications will the affected files be created anew.

While the git.kernel.org (g.k.o) is the canonical home for the kernel, that server is slow for our download. We anyways won't be pushing upstream directly. So, it's best to use the googlesource mirror.

Clone the mainline once in a folder named `updated_mainline` as follows:

```shell
$ git clone https://kernel.googlesource.com/pub/scm/linux/kernel/git/torvalds/linux updated_mainline
```

Now whenever you need to do some work, let's say work on a UAF in net, you can simply do either of the following:

```shell
# Worktree method (inside updated_mainline).
$ git worktree add ../net_uaf
$ cd ../net_uaf

# Copy method.
$ cp -r updated_mainline/ net_uaf
$ git switch -c uaf_fix    # Do not work on master branch.
```

While `git worktree` should be preferred, I use the copy method, because that's what I started with :-P.

---

## Setting up git

One must ensure that the git configuration is correctly set for kernel repos, as identity is very crucial for signing your commits and sending them via email. 

```shell
# Set identity.
$ git config user.name "Your name"
$ git config user.email "your@email.com"
```

I have GPG signing enabled globally in git, but kernel commits for plebs like us must not be GPG signed, it is used only by maintainers.

GPG signing is done, on say GitHub, to ensure that the git identity is not fake. By adding a public key to GitHub and checking the signatures in commits, GitHub can verify committer identity. But in kernel development, we use mailing list, so we actually do not need GPG signing. The `Signed-off-by` email and the `From:` address of the patch email must match.

```shell
# Disable GPG signing.
$ git config commit.gpgsign false
```

Upon pulling, rebase should be done. The upstream is supreme. Also, whenever someone says on mailing list to rebase, you just need to pull.

```shell
# Rebase on pull.
$ git config pull.rebase true
```

Email SMTP details should be set properly, as the patches to the mailing lists are typically sent using `git send-email`. But please

-   **DO NOT store the password in config**. One can utilise keyrings to store the password.  Here is an excellent blog post detailing the process: [Using git send-email with gnome-keyring – Guillaume Champagne](https://gchamp20.github.io/git-send-email-gnome-keyring/) (web archive [here](https://web.archive.org/web/20231012164752/https://gchamp20.github.io/git-send-email-gnome-keyring/)).
-    **If possible, make sure to create an application specific password to use for git.**

This all then simply results in the following git commands:

```shell
# Set up email settings for send-email.
$ git config sendemail.smtpServer "smtp.email.com"
$ git config sendemail.smtpEncryption tls
$ git config sendemail.smtpServerPort 587
$ git config sendemail.smtpUser "your@email.com"
$ git config sendemail.confirm "always"
```

It's effectively mandatory to use `checkpatch.pl` script to check your patches, otherwise you'll have complaints over the mailing list (outside of genuine cases). Manually doing it is tiring, and one may forget to do it or get tired of it.

Fortunately, git allows for hooks to be executed before and after actions like commit. Whenever someone is annoying you with web3 BS like blockchain with smart contracts, just show them git.

Shuah in the LFD103 course[^lfd103] suggests a neat trick to add checkpatch to the post-commit hook. Basically, after committing changes you extract the commit in patch format (using `git show`) and pass it to checkpatch, which will show whether there are any problems or everything is okay.

[^lfd103]: [A Beginner's Guide to Linux Kernel Development (LFD103) - The Linux Foundation](https://trainingportal.linuxfoundation.org/courses/a-beginners-guide-to-linux-kernel-development-lfd103)

To do that, add the following in `updated_mainline/.git/hooks/post-commit` and make it executable:

```bash
#!/bin/bash

patch=$(git show --format=email HEAD)
./scripts/checkpatch.pl --show-types --strict --codespell - <<< "$patch"
```

### kgit_config

We can put everything together in a bash function as follows:

```bash
kgit-config() {
	git config user.name "Your name"
	git config user.email "your@email.com"
	git config commit.gpgsign false
	git config pull.rebase true
	
	git config sendemail.smtpServer "smtp.email.com"
	git config sendemail.smtpEncryption tls
	git config sendemail.smtpServerPort 587
	git config sendemail.smtpUser "your@email.com"
	git config sendemail.confirm "always"
	
	echo -e '#!/bin/bash
		
		patch=$(git show --format=email HEAD)
		./scripts/checkpatch.pl --show-types --strict --codespell - <<< "$patch"
	' | awk '{$1=$1};1' > .git/hooks/post-commit
	chmod u+x .git/hooks/post-commit
}
```

Now, you can just use `kgit-config` on terminal and have everything done for you.

Even though this is a one-time setup (because you use/copy this repo in the end), it's good to have this alias. You may also work on subsystem-specific trees, linux-next, etc., mainline isn't the only repo.

---

## Setting up QEMU image

This section is essentially from the syzkaller docs on setting up QEMU[^syzkaller-qemu]. I will be talking about x86-64.

[^syzkaller-qemu]: https://github.com/google/syzkaller/blob/master/docs/linux/setup_ubuntu-host_qemu-vm_x86-64-kernel.md

Make a specific folder for storing QEMU related things, like the system image. It's good to organise things.

```shell
$ mkdir -p qemu/image
$ cd qemu/image
```

We will use the `create-image.sh` script provided by Google for syzkaller. It will create a Debian image with a passwordless root user, and also generate SSH keys so that we can use SSH to login into the system once run by QEMU.

Download the script as follows:

```shell
$ wget https://raw.githubusercontent.com/google/syzkaller/master/tools/create-image.sh
$ chmod +x create-image.sh
```

You may want to remove the `-x` flag from line 7 in the script to avoid spam during runtime.

Create a 10 GB image (suffiicient size to install some things later and not worry about space) containing Debian 12 (bookworm) as follows:

```shell
$ ./create-image.sh -s 10240 -d bookworm -f full

# Delete leftover chroot folder from building
$ sudo rm -rf chroot
```

Name the image `bookworm.img` so you know what it is, and the associated SSH keys similarly. They all reside in the same folder as the script, here that is `qemu/image`.

---

## Configuring the kernel

One can generate a default config with `make defconfig`.

Though, generally when debugging bugs, like those listed on syzkaller, you'll get a config file (which is to be stored as `.config` in the kernel root). These may not have all the new config options, and during build time it may ask you for the values those options should have. You could just spam (press and hold) the enter key to set the defaults, or use the handy `merge_config.sh` script as:

```shell
$ ./scripts/kconfig/merge_config.sh .config
```

But this script is for a more powerful purpose. As the name suggests, it merges the config files automatically. It accounts for all dependencies for you, so you do not have to bother with setting values yourself normally.

This allows for a neat way to configure kernel for our QEMU machines because we need to set some more options. It would be tiring to do them by hand, so we can just have the relevant options in a config file somewhere, let's say in `$HOME/linux/config`, and then pass it to this script.

For our QEMU machine, need to include the `kvm_guest.config` from `kernel/configs` in the repo, as that includes some necessary config options, like enabling virtIO, 9P filesystem, networking, etc.

We need some options like `CONFIGFS_FS`, `SECURITYFS` enabled for correctly booting our image. Also, debugging benefits from a couple of other things, like KASAN, UBSAN, preemption, lockup detector, etc. So, we can have a common config as follows, which we will always include.

{{< code language="config" title="common.config" isCollapsed="true" >}}
CONFIG_DYNAMIC_DEBUG=y
CONFIG_DEBUG_FS=y
CONFIG_CONFIGFS_FS=y
CONFIG_SECURITYFS=y

CONFIG_DEBUG_INFO=y

CONFIG_KASAN=y
CONFIG_UBSAN=y

CONFIG_PANIC_ON_OOPS=y
CONFIG_PANIC_ON_OOPS_VALUE=1
CONFIG_PANIC_TIMEOUT=0


CONFIG_PREEMPT_BUILD=y
CONFIG_PREEMPT=y
CONFIG_PREEMPT_COUNT=y
CONFIG_PREEMPTION=y
CONFIG_PREEMPT_DYNAMIC=y
CONFIG_SCHED_CORE=y
CONFIG_PREEMPT_RCU=y
CONFIG_PREEMPT_NOTIFIERS=y

CONFIG_LOCKUP_DETECTOR=y

CONFIG_SOFTLOCKUP_DETECTOR=y
CONFIG_BOOTPARAM_SOFTLOCKUP_PANIC=y

CONFIG_HAVE_HARDLOCKUP_DETECTOR_BUDDY=y
CONFIG_HARDLOCKUP_DETECTOR=y
CONFIG_HARDLOCKUP_DETECTOR_PERF=y
CONFIG_HARDLOCKUP_DETECTOR_COUNTS_HRTIMER=y
CONFIG_HARDLOCKUP_CHECK_TIMESTAMP=y
CONFIG_BOOTPARAM_HARDLOCKUP_PANIC=y

CONFIG_DETECT_HUNG_TASK=y
CONFIG_DEFAULT_HUNG_TASK_TIMEOUT=140
CONFIG_BOOTPARAM_HUNG_TASK_PANIC=y
CONFIG_WQ_WATCHDOG=y

CONFIG_TUN=y


CONFIG_FAULT_INJECTION=y
CONFIG_FAILSLAB=y
CONFIG_FAIL_PAGE_ALLOC=y
CONFIG_FAULT_INJECTION_USERCOPY=y
CONFIG_FAIL_MAKE_REQUEST=y
CONFIG_FAIL_IO_TIMEOUT=y
CONFIG_FAIL_FUTEX=y
CONFIG_FAULT_INJECTION_DEBUG_FS=y
{{< /code >}}

Sometimes we may need more options which are there in `kernel/configs/debug.config` in the kernel root. Also, we may sometimes want to enable tracing or options like 9P debugging, which can be done by adding the options in another config file as follows:

{{< code language="config" title="tracing.config" isCollapsed="true" >}}
CONFIG_KCOV=y
CONFIG_KASAN=y
CONFIG_KASAN_INLINE=y
CONFIG_FTRACE=y
CONFIG_DYNAMIC_FTRACE=y
CONFIG_FUNCTION_TRACER=y
CONFIG_FUNCTION_GRAPH_TRACER=y
CONFIG_IRQSOFF_TRACER=y
CONFIG_PREEMPT_TRACER=y
CONFIG_SCHED_TRACER=y
CONFIG_STACK_TRACER=y
CONFIG_BLK_DEV_IO_TRACE=y
CONFIG_FUNCTION_PROFILER=y
CONFIG_FTRACE_MCOUNT_RECORD=y

CONFIG_NET_9P_DEBUG=y
{{< /code >}}

Now, we can simply create our config as:

```shell
$ ./scripts/kconfig/merge_config.sh .config \
	./kernel/configs/kvm_guest.config \
	$HOME/linux/config/common.config
```

For the extra debug options, simply add the config files:

```shell
$ ./scripts/kconfig/merge_config.sh .config \
	./kernel/configs/kvm_guest.config \
	./kernel/configs/debug.config \
	$HOME/linux/config/common.config \
	$HOME/linux/config/tracing.config
```

### kconfig

We can set up a bash function for this as follows:

```bash
kconfig() {
	case "$1" in
		"--def")
			make defconfig "${@:5}"
			./scripts/kconfig/merge_config.sh .config \
				./kernel/configs/kvm_guest.config \
				$HOME/linux/config/common.config
			;;

		"--fix")
			./scripts/kconfig/merge_config.sh .config \
				./kernel/configs/kvm_guest.config \
				$HOME/linux/config/common.config
			;;

		"--debug")
			./scripts/kconfig/merge_config.sh .config \
				./kernel/configs/kvm_guest.config \
				./kernel/configs/debug.config \
				$HOME/linux/config/common.config \
				$HOME/linux/config/tracing.config
			;;

		*)
			echo "Pass --def, --fix, or --debug."
			;;
	esac
}
```

Now we can just quickly configure using `kconfig` command.

My gist has more options, if interested one can check it out.

---

## Building the kernel

The kernel can be built with:

```shell
$ make -j`nproc` all
```

The `nproc` allows for parallelised building, so that the compile time is significantly improved. Some people suggest to use a number = 1.5*nproc in the argument (like if nproc is 12, use -j18).

Sometimes we may want to build with LLVM. For that we can pass `LLVM=1` in the make argument, as follows:

```shell
$ make -j`nproc` all LLVM=1
```

Now, if we use QEMU to run the kernel, it will be successful, but you'll notice that any loadable modules you may have enabled or made won't be there, because the loadable `.ko` files, or even the `/lib/modules` folder, simply does not exist.

We need to use the `modules_install` make recipe to copy them to a location, and somehow access it inside the VM. We can achieve that using virtio 9P filesystem. That's why I had mentioned we need to enable it during the configuration.

Let's copy the modules to a folder named `.compiled_modules` in the tree root. Note that the name starts with a dot, because the `.gitignore` is configured to ignore most hidden files and folders.

```shell
$ make INSTALL_MOD_PATH=./.compiled_modules modules_install
```

`modules_install` creates a symlink to the kernel source tree in `/lib/modules/<version>/build`, but we cannot really access the symlink from inside the VM (as the mount point will be the `.compiled_modules` folder, and not its parent).

So please delete that symlink. One can copy the entire tree to the `build` directory, but there is no need to. But it is a good idea to copy the config file, so the config of the currently running kernel is accessible inside the VM.

### kmake-all

We can now put everything in a bash function as follows.

```bash
kmake-all() {
	(
		set -e

		if [[ -f ".llvm" ]]; then
			echo -e ".llvm found - Using LLVM.\n"
			make -j18 all LLVM=1 "$@"
		else
			echo -e ".llvm not found - Using GCC\n"
			make -j18 all "$@"
		fi

		# Store loadable modules so we can use in QEMU via virtfs.
		make INSTALL_MOD_PATH=./.compiled_modules modules_install

		# Symlink to local dir in /lib/modules/*/build won't work in
		# the VM, so remove it and just add the config file there.
		rm ./.compiled_modules/lib/modules/*/build
		mkdir $(ls -d ./.compiled_modules/lib/modules/*)/build
		cp .config ./.compiled_modules/lib/modules/*/build
	)
}
```

Note that we automated the LLVM argument by checking if a file named `.llvm` exists in the repo root. You can make using `touch .llvm`. This is done because it is annoying to write `LLVM=1` everytime, it's very convenient to just type `km` and use tab completion.

---

## Running the kernel in QEMU (you are probably here for this)

The most confusing part most people beginning with kernel development have is running QEMU properly.

QEMU has a ***lot*** of arguments listed in its manpage. Naturally, it might look like a daunting task at first to figure it out, but after done once it actually starts making perfect sense.

### Explanation of the options we use

Since we are going to emulate an x86-64 system, the program name would be `qemu-system-x86_64`.

As we are on an x86-64 CPU already, we emulate the same CPU using the `-cpu host` option.

We enable KVM virtualisation acceleration using `-accel kvm`, which allows for the near native performance.

We are going to work using the command line, so we disable the graphics with `-nographics` option.

We can allocate memory to the VM using `-m` option. For instance, if we want to allocate 4 GB of RAM, we will use `-m 4G`.

We can allocate the number of cores using the `-smp` option. This is important, as race conditions will not happen without multicore setup, as you'll only have one thread running by default (`-smp 1`). I allocate half the number listed by `nproc`. For example, `-smp 6`.

We need to specify the kernel image location using `-kernel` flag. If in the kernel repo root after building, use `-kernel ./arch/x86_64/boot/bzImage`.

We can pass command-line parameters for the kernel using the `-append` option.

The `console=ttyS0` param tells the kernel to dump the logs (same as `dmesg`) on ttyS0, which is the tty emulated by the QEMU window. The `root=/dev/sda` param tells the root partition of the image is `/dev/sda`. The `earlyprintk=serial` param tells the kernel to print the early boot messages over the serial port. The `net.ifnames=0` param tells the kernel to disable the Predictable Network Interface Names mechanism, which is needed if the "Failed to start Raise network interfaces" error is encountered[^syzkaller-troubleshooting]. The `nokaslr` param disables Kernel Address Space Layout Randomization (KASLR) and causes kernel to be loaded in its standard memory location. The `selinux=1 (or 0)` param enables/disables the SELinux module.

[^syzkaller-troubleshooting]: https://github.com/google/syzkaller/blob/master/docs/linux/troubleshooting.md

The arguments discussed above can be passed using `-append "console=ttyS0 root=/dev/sda earlyprintk=serial net.ifnames=0 nokaslr selinux=0"`.

We need to specify the location of our raw format Debian image we created using the `-drive` option. For example, `-drive file=$HOME/linux/qemu/image/bookworm.img,format=raw`. Note the lack of space between the two kwargs, it is important so that QEMU parses it correctly as arguments to `-drive`.

We can set the networking options using `-net` option.

We need to set SSH settings, as we discusses we allow for SSH login. For our case, we will set them using `-net user,host=10.0.2.10,hostfwd=tcp:127.0.0.1:10021-:22`.

We also set the emulated NIC settings using `-net nic,model=e1000`.

We can store the PID of the QEMU VM using `-pidfile` option. For example, `-pidfile vm.pid`.

We also mentioned we access the `.compiled_modules` inside the VM. We do that by setting a mount point for the 9p virtfs using the `-virtfs` option.[^qemu-9p] Specifically, we use the following (remember the lack of space after commas):

[^qemu-9p]: [Documentation/9psetup - QEMU](https://wiki.qemu.org/Documentation/9psetup)

```
-virtfs local,path=./.compiled_modules/lib/modules,mount_tag=compiled_modules,security_model=none
```

The first argument specifies the filesystem driver. We simply use `local` so the individual virtual FS functions are translated directly to act on the host filesystem. The `path` specifies the path of the directory to mount (which in our case is the directory we want to be mounted to `/lib/modules` in the VM). `mount_tag` specifies the tag of the mount point inside the VM. The `security-model` specifies the security model for the files in path (credentials, attributes, etc.). We use `none` so files created in VM are stored as it is. You can read more on the QEMU docs.[^qemu-9p]

Lastly, we can store all the output by piping the QEMU output to `tee` command.

### The command

Therefore, to run our compiled kernel on QEMU, we use the following command by putting together everything:

```shell
$ qemu-system-x86_64 \
	-cpu host \
	-accel kvm \
	-nographic \
	-m 4G \
	-smp 6 \
	-kernel ./arch/x86_64/boot/bzImage \
	-append "console=ttyS0 root=/dev/sda earlyprintk=serial net.ifnames=0 nokaslr selinux=0" \
	-drive file=$HOME/linux/qemu/image/bookworm.img,format=raw \
	-net user,host=10.0.2.10,hostfwd=tcp:127.0.0.1:10021-:22 \
	-net nic,model=e1000 \
	-pidfile vm.pid \
	-virtfs local,path=./.compiled_modules/lib/modules,mount_tag=compiled_modules,security_model=none \
2>&1 | tee vm.log
```

### Logging in

***The username to login is `root`*** if you followed the mentioned procedure to make the image.

To login using SSH, in a new window use the following SSH command (which uses the key we generated):

```shell
$ ssh -i $HOME/linux/qemu/image/bookworm.id_rsa -p 10021 -o "StrictHostKeyChecking no" root@localhost
```

### Terminating

Use Ctrl+A and then X to terminate the VM.

### kemulate

We can put everything together, and allow for more user arguments which can override the defaults, in a bash function / aliases as follows:

{{< code language="bash" title="kemulate" isCollapsed="true" >}}
kemulate() {
	qemu_args=""

	# We will pass user args to QEMU, except the flags.
	# Since flag processing is needed, we will append them in the end.
	# Also, by appending the user args at the end, the default arguments
	# can be overriden by user.
	user_args="$@"
	
	# Use host for the KVM on terminal.
	qemu_args+="-cpu host -accel kvm -nographic"
	qemu_args+=" "
	
	# Memory size.
	qemu_args+="-m 4G"
	qemu_args+=" "
	
	# Number of cores.
	qemu_args+="-smp 6"
	qemu_args+=" "
	
	# Kernel image location.
	qemu_args+="-kernel ./arch/x86_64/boot/bzImage"
	qemu_args+=" "
	
	# Kernel command line parameters.
	qemu_args+="-append \"console=ttyS0 root=/dev/sda earlyprintk=serial"
	qemu_args+=" net.ifnames=0 nokaslr"
	
	if [[ "$1" == "--selinux" ]]; then
		qemu_args+=" selinux=1"
		user_args="${user_args:9}"
	else
		qemu_args+=" selinux=0"
	fi
	
	qemu_args+="\" "
	
	# Debian image to boot.
	qemu_args+="-drive file=$HOME/linux/qemu/image/bookworm.img,"
	qemu_args+="format=raw"
	qemu_args+=" "
	
	# User settings for SSH.
	qemu_args+="-net user,host=10.0.2.10,hostfwd=tcp:127.0.0.1:10021-:22"
	qemu_args+=" "
	
	# NIC settings.
	qemu_args+="-net nic,model=e1000"
	qemu_args+=" "
	
	# PID file.
	qemu_args+="-pidfile vm.pid"
	qemu_args+=" "
	
	# Use virtfs to get access to loadable modules inside QEMU.
	qemu_args+="-virtfs local,path=./.compiled_modules/lib/modules,"
	qemu_args+="mount_tag=compiled_modules,security_model=none"
	qemu_args+=" "
	
	# Append the user args.
	qemu_args+="$user_args"
	
	# Run QEMU (eval used to avoid bash splitting arguments on spaces)
	eval "qemu-system-x86_64 $qemu_args 2>&1 | tee vm.log"
}


alias kemulate-one="kemulate -smp 1"
{{< /code >}}

Now, just use `kemulate` to run the kernel. You can use `kemulate-one` if you don't want multi-core setup. You can use `--selinux` if you want to enable selinux. You can, as mentioned earlier, add and override the QEMU arguments by passing them to the function.

### klogin

Similarly, we can make a bash function for logging in via SSH (and store the output to a file) as follows:

```bash
klogin() {
	ssh -i $HOME/linux/qemu/image/bookworm.id_rsa \
		-p 10021 -o "StrictHostKeyChecking no" root@localhost \
	2>&1 | tee ssh_op.log
}
```

---

## Editing fstab to mount /lib/modules inside the VM image

We booted into the VM, but we still have not mounted the 9P mount point we have created for `/lib/modules`.

We can use the command line to do it, but doing it everytime is a hassle. So, we just add an entry in `/etc/fstab` so that it is automatically mounted at boot.

Append the following in `/etc/fstab` and save:

```
# <file system>		<mount pt>			<type>		<options>			<dump>	<pass>
compiled_modules	/lib/modules			9p		trans=virtio,version=9p2000.L	0	0
```

Now restart the VM (quit and `kemulate` again), and see if the folder in `/lib/modules` exist. If you followed everything correctly till now, it will be there.

---

## Decoding stacktrace

Sometimes you'll want to decode a stacktrace dumped by the kernel. We can use the `decode_stacktrace.sh` script in the kernel `scripts` folder for it.

The following bash function does it for you:

```bash
ktrace-decode() {
	if [[ "$1" == "" || "$2" == "" ]]; then
		echo "Provide input and output filenames."
		exit 1
	fi

	(./scripts/decode_stacktrace.sh vmlinux < "$1") > "$2"
}
```

Use as `ktrace-decode in.log out.log`. 

Many times you can make `in.log` by appropriately trimming the `vm.log` file we tee'd QEMU's output to in our command.

---

## Getting maintainers

You debugged everything and created patches. Now who to send these patches to? 

One can use `./scripts/get_maintainer.pl` to get the appropriate people and mailing lists to send the patch to.

You can use the script name directly. It is big enough for me to create a simple bash alias, so here it is:

```bash
kget-maintainers() {
	./scripts/get_maintainer.pl "$@"
}
```

One can extend this into `ksend-patch` which can parse the `get_maintainer.pl` output and appropriately sets the `to` and `cc` arguments for `git send-email`.

---

That's it for now. Hope you found this useful!
