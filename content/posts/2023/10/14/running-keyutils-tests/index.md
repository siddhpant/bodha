---
title: Running keyutils tests
tags: ["kernel", "debugging", "tests", "keyutils", "linux"]
---

It's not very apparent how to run the tests in the keyutils repo ([kernel/git/dhowells/keyutils.git - Key management utilities](https://git.kernel.org/pub/scm/linux/kernel/git/dhowells/keyutils.git)), especially if you don't work on kernel areas like keyrings and watch queue.

Running tests does require some kernel preparation. Searching on internet fetches you limited information. So I looked further what would cause some failures by looking at the test code.

I was able to run all the test using the following config options enabled (you can paste in a file and then use the `merge_config` script, see my previous post):

```config
CONFIG_KEYS=y
CONFIG_TMPFS=y
CONFIG_CRYPTO_LIB_CHACHA20POLY1305=y
CONFIG_BIG_KEYS=y

CONFIG_TRUSTED_KEYS=y
CONFIG_SYSTEM_TRUSTED_KEYRING=y
CONFIG_SECONDARY_TRUSTED_KEYRING=y
CONFIG_SYSTEM_BLACKLIST_KEYRING=y

CONFIG_SYSTEM_TRUSTED_KEYS="key.pem"

CONFIG_CRYPTO_DH=y
CONFIG_KEY_DH_OPERATIONS=y

CONFIG_INTEGRITY=y
CONFIG_INTEGRITY_SIGNATURE=y
CONFIG_INTEGRITY_ASYMMETRIC_KEYS=y
CONFIG_INTEGRITY_TRUSTED_KEYRING=y
```

Note that we specify a key named `key.pem` to `CONFIG_SYSTEM_TRUSTED_KEYS`, which causes the key to be added in the trusted keyring during compile time.

We can generate a dummy key for tests using the following command:
```shell
$ openssl req -x509 -newkey rsa:4096 -out key.pem -sha256 -nodes -subj "/CN=."
```

Now build the kernel and boot into it using QEMU.

Install the keyutils packages (on Debian) as follows:

```shell
$ sudo apt install keyutils libkeyutils*
```

Clone the keyutils repo:

```shell
$ git clone https://kernel.googlesource.com/pub/scm/linux/kernel/git/dhowells/keyutils.git
$ cd keyutils
```

Now run tests using `make`:

```shell
$ make test
```

Now all tests in the `tests/` directory will run.
