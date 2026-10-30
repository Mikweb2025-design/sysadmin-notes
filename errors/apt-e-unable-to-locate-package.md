# apt: E: Unable to locate package

> apt does not find the package in any package list it knows. Either the lists are empty or old, the name is wrong for your release, or the repository that carries the package is not enabled.

Example:

```
$ sudo apt install htop
Reading package lists... Done
Building dependency tree... Done
E: Unable to locate package htop
```

## Common causes

- `apt update` was never run, for example on a fresh server or in a new container, so the package lists are empty.
- The package name is wrong, or the package has another name on your release.
- The repository component that carries the package is not enabled: `universe` on Ubuntu, `contrib` or `non-free` on Debian.
- The release is end of life and its repositories moved to an archive server, so `apt update` ends with 404 errors.
- A broken or typed-wrong entry in the sources makes `apt update` fail, so the lists stay incomplete.

## How to check

Does apt know the package at all?:

```bash
apt-cache policy htop
```

Search for the real package name:

```bash
apt-cache search htop | head -n 10
```

Which repositories and components are active?:

```bash
grep -rE '^(deb |Types:|URIs:|Suites:|Components:)' /etc/apt/sources.list /etc/apt/sources.list.d/ 2>/dev/null
```

Which release is this?:

```bash
cat /etc/os-release
```

## Fix

1. Refresh the lists and read the output for errors: `sudo apt update`.
2. Use the exact package name that `apt-cache search` shows for your release.
3. Enable the missing component: on Ubuntu `sudo add-apt-repository universe`, on Debian add `contrib` or `non-free` to the `Components` of your sources, then run `apt update` again.
4. If `apt update` shows 404 for your release, it is end of life. Upgrade to a supported release instead of pointing the sources to an archive for production.
5. In a Dockerfile, run `apt-get update` and `apt-get install` in the same `RUN` step.

## FAQ

**Why does it work on my laptop but not in the container?**

Container images ship without package lists to stay small. Run `apt-get update` first, in the same build step as the install.

**Is it safe to add third-party repositories to fix this?**

Only if you trust the source. First check whether the package exists under another name or in an official component of your release.

---
More detail, related guides and tools: **[daniele-michelin.com/en/errors/apt-e-unable-to-locate-package/](https://daniele-michelin.com/en/errors/apt-e-unable-to-locate-package/)**
