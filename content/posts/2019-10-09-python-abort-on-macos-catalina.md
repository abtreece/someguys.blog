---
layout: post
title: Python Abort on MacOS Catalina
date: 2019-10-09
tags: ["python", "mac"]
---

Yes. I upgraded to Catalina on the first day. `¯\_(ツ)_/¯`

Now I'm trying to run a Python program and it's exiting with `Abort trap: 6`. The crash report indicates the specific problem is with an OpenSSL dylib file...<!--more-->

```
Application Specific Information:
/usr/lib/libcrypto.dylib
abort() called
Invalid dylib load. Clients should not load the unversioned libcrypto dylib as it does not have a stable ABI.
```

This may be a thing that gets worked out by an Xcode or Homebrew update, but until that time the simple fix is to symlink a couple files:

```
$ ln -s /usr/local/Cellar/openssl/1.0.2t/lib/libcrypto.1.0.0.dylib /usr/local/lib/libcrypto.dylib
$ ln -s /usr/local/Cellar/openssl/1.0.2t/lib/libssl.1.0.0.dylib /usr/local/lib/libssl.dylib
```

Obviously, I'm using the latest OpenSSL from Homebrew which you can ensure you have with:

```
$ brew install openssl
```

Carry on.
