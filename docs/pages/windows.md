Both Allzpark and Rez are cross-platform, but each platform has a few gotchas to keep in mind. Here's a quick primer on how to make the most out of Allzpark and Rez on the Windows operating system.

<br>

## Long File Paths

Windows has a max path length of 260 characters, which can become an issue for packages on a long repository path and multiple variants.

<br>

### Problem

```bash
# Repository root
\\mylongstudioaddress.local\main\common\utilities\packages\internal

# Package
\maya_essentials\1.42.5beta\platform-windows\arch-AMD64\os-windows-10.0.1803

# Payload
\python\maya_essentials\utilities\__init__.py
```

**188 characters**

That's a relatively common path to a Python package, packaged with Rez, and we're already close to the 260 character limit. Now take backslashes into account, and that Python and friends escape those prior to using them. There are 16 backslashes in there, which adds another 16 characters.

```bash
# Before
\long\path

# After
\\long\\path
```

**204 characters**

We still haven't changed the path, and yet the length has increased. Now take into account some libraries taking extra precautions and escapes even estaped backslashes.

```bash
# Before
\\long\\path

# After
\\\\long\\\\path
```

That adds yet another 32 characters.

**236 characters**

And again, we haven't changed our path, and yet this is what some tools will be working with, leaving you with very little room.

<br>

### Solution

You've got at least three options here.

1. Patch your paths
1. Patch Rez
2. Patch Windows

**Patch Paths**

The most straightforward, but likely difficult, thing to do is to avoid long paths altogether.

- Use a short hostname
- Use a short repository path
- Abbreviate Python libraries
- Don't use Python packages from PyPI with long names

But a lot of this is not practical, and merely postpones the issue.

**Patch Rez**

I've investigated what it would take to make changes to Rez that facilitate longer paths, and found that there is a prefix you can use for paths that will "force" Windows to interpret paths longer than 260 characters.

```bash
# Before
c:\long\path.exe

# After
\\?\c:\long\path.exe
```

Since paths are entirely managed by Rez, it wouldn't be unreasonable to wrap any path creation call to prefix the results with `\\?\` if the user was running Windows. But I couldn't find a single-point-of-entry for these, as paths were generated all over the place. Rightly so; it would be borderline overengineering to wrap all calls to e.g. `os.path.join` or `os.getcwd` into a "prefixer" just for this occasion. It would however have helped in this particular case.

Furthermore, this would only really apply to Windows 10 and above, since from what I gather this (poorly documented) feature is only available there; possibly related to this next feature.

**Patch Windows**

You wouldn't think this is an option, but it just might be.

This technically doesn't count as patching Windows, but because we're changing a fundamental component of the OS - something each applications has till now taken for granted - it may cause all sorts of havok for applications that depend on the 260 character limit.

> Relevant comic https://xkcd.com/1172/

Since June 20th 2017, users of Windows 10 1607 have had the ability to enable support for "long paths".

```powershell
# From an administrator PowerShell session
Set-ItemProperty -Path HKLM:\SYSTEM\CurrentControlSet\Control\FileSystem -Name LongPathsEnabled -Value 1 -Type DWord
```

This would effectively prepend `\\?\` to every path "under the hood", solving the issue. But at what cost?

Let the community know if you encounter any issues by making [an issue](https://github.com/mottosso/allzpark/issues/new).

<br>

## Process Tree

Virtualenv is one way of using Rez on Windows, and if you do then the `rez.exe` executable is generated during `pip install` and works by spawning a `python.exe` process, also generated by `pip`, which in turn calls on your system `python.exe`. Here's what spawning your own Python session from within a Rez context looks like.

![image](https://user-images.githubusercontent.com/2152766/59964221-e6b48780-94f5-11e9-8300-390d0587f5e3.png)

<br>

## Maya and Quicktime

Typically, playblasting to `.mp4` or `.mov` with Maya requires a recent install of Quicktime on the local machine. Let's have a look at how to approach this with Rez.

> How *does* one approach this with Rez? Submit [a PR](https://github.com/mottosso/allzpark) today!

<br>