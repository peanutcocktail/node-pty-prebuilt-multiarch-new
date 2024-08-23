<p align="center">
  <a href="https://homebridge.io"><img src="https://raw.githubusercontent.com/homebridge/branding/latest/logos/homebridge-color-round-stylized.png" height="140"></a>
</p>
<span align="center">

# node-pty-prebuilt-multiarch

[![npm](https://badgen.net/npm/v/@homebridge/node-pty-prebuilt-multiarch/latest)](https://www.npmjs.com/package/@homebridge/node-pty-prebuilt-multiarch)
[![npm](https://badgen.net/npm/dt/@homebridge/node-pty-prebuilt-multiarch?label=downloads)](https://www.npmjs.com/package/@homebridge/node-pty-prebuilt-multiarch)
![Prebuild Binaries](https://github.com/homebridge/node-pty-prebuilt-multiarch/workflows/Build%20and%20Test/badge.svg)
[![Discord](https://badgen.net/discord/online-members/C87Pvq3?icon=discord&label=discord)](https://discord.gg/C87Pvq3)

</span>

This project is a parallel fork of [`node-pty`](https://github.com/Microsoft/node-pty) providing prebuilt packages for certain Node.js and Electron versions.

Inspired by [daviwil/node-pty-prebuilt](https://github.com/daviwil/node-pty-prebuilt).

## Usage

Thanks to the excellent [`prebuild`](https://github.com/prebuild/prebuild), [`prebuild-install`](https://github.com/prebuild/prebuild) modules, and [`prebuildify`](https://github.com/prebuild/prebuildify) using this module is extremely easy.
You merely have to change your `node-pty` dependency to `@homebridge/node-pty-prebuilt-multiarch` and then change any `require` statements in your code from `require('node-pty')` to `require('@homebridge/node-pty-prebuilt-multiarch')`.

> **NOTE**: We started shipping prebuilds as of node-pty version 0.8.1, no prior versions are provided!
> If you were using an earlier version of `node-pty` you will need to update your code to account for any API changes that may have occurred.

## How It Works

We maintain a parallel fork of the `node-pty` codebase that will be updated as new releases are shipped.
When we merge new updates to the code into the `prebuilt-multiarch` branch, new prebuilt packages for our supported Node.js and Electron versions are updated to the corresponding [GitHub release](https://github.com/homebridge/node-pty-prebuilt-multiarch/releases).

When `@homebridge/node-pty-prebuilt-multiarch` is installed as a package dependency, the installation script checks to see if there's a prebuilt package on this repo for the OS, ABI version, and architecture of the current process and then downloads it, extracting it into the module path.
If a corresponding prebuilt package is not found, `node-gyp` is invoked to build the package for the current platform.

## Prebuilt Versions

| OS            | Architectures             |
|---------------|---------------------------|
| macOS         | x64, arm64                |
| Linux (glibc) | ia32, x64, armv6, aarch64 |
| Linux (musl)  | x64, armv6, aarch64       |
| Windows       | ia32, x64                 |

We only provide prebuilt binaries for Node.js 16 and Electron 16.0.0 or higher.  Pls note that prebuilds for Electron 28 are not supplied due to build issues.

## Build / Package

Please note releasing this package uses GitHub actions.

This flows takes the branch selected from the workflow start drop down, and creates a GitHub and NPM Release containing the prebuild artifacts.
The version of the Release comes from the package.json, and in the case of a BETA release automatically appends the beta release version.
During processing, it leverages a branch called `release-candidate` as a holding area for prebuilds.

When running the job, most times a couple of the instances of the sub step `Commit & Push Changes` within `Prebuild NPM and GitHub Release artifacts` fails.
When this occurs just re-run. This is due to concurrency issues between the steps and GitHub.
A typical run has 3-4 steps fail.

1. Create branch `release-candidate` if not existing (the script deletes it before starting and will fail if it isn't present)
2. Ensure version tag within package.json reflects version you want to publish, please note beta tags are added by the action.
3. Run Action `Run prebuild's and Create GitHub and NPM release`, and select branch you wish to publish, and if it needs to be BETA tagged and versioned
4. This will run for about an hour, and create a GitHub release with the prebuild artifacts attached, and a npm release with the prebuild artifacts attached

```bash
# Install dependencies and build C++
npm install
# Compile TypeScript -> JavaScript
npm run build
```

## Dependencies

Node.JS 16 or Electron 19 is required to use `node-pty`. What version of node is supported is currently mostly bound to [whatever version Visual Studio Code is using](https://github.com/microsoft/node-pty/issues/557#issuecomment-1332193541).

### Linux (apt)

```sh
sudo apt install -y make python build-essential
```

### macOS

Xcode is needed to compile the sources, this can be installed from the App Store.

### Windows

`npm install` requires some tools to be present in the system like Python and C++ compiler. Windows users can easily install them by running the following command in PowerShell as administrator. For more information see https://github.com/felixrieseberg/windows-build-tools:

```sh
npm install --global --production windows-build-tools
```

The following are also needed:

- [Windows SDK](https://developer.microsoft.com/en-us/windows/downloads/windows-10-sdk) - only the "Desktop C++ Apps" components are needed to be installed
- Spectre-mitigated libraries - In order to avoid the build error "MSB8040: Spectre-mitigated libraries are required for this project", open the Visual Studio Installer, press the Modify button, navigate to the "Individual components" tab, search "Spectre", and install an option like "MSVC v143 - VS 2022 C++ x64/x86 Spectre-mitigated libs (Latest)" (the exact option to install will depend on your version of Visual Studio as well as your operating system architecture)

## Debugging

[The wiki](https://github.com/Microsoft/node-pty/wiki/Debugging) contains instructions for debugging node-pty.

## Security

All processes launched from node-pty will launch at the same permission level of the parent process. Take care particularly when using node-pty inside a server that's accessible on the internet. We recommend launching the pty inside a container to protect your host machine.

## Thread Safety

Note that node-pty is not thread safe so running it across multiple worker threads in node.js could cause issues.

## Flow Control

Automatic flow control can be enabled by either providing `handleFlowControl = true` in the constructor options or setting it later on:

```js
const PAUSE = '\x13';   // XOFF
const RESUME = '\x11';  // XON

const ptyProcess = pty.spawn(shell, [], {handleFlowControl: true});

// flow control in action
ptyProcess.write(PAUSE);  // pty will block and pause the child program
...
ptyProcess.write(RESUME); // pty will enter flow mode and resume the child program

// temporarily disable/re-enable flow control
ptyProcess.handleFlowControl = false;
...
ptyProcess.handleFlowControl = true;
```

By default `PAUSE` and `RESUME` are XON/XOFF control codes (as shown above). To avoid conflicts in environments that use these control codes for different purposes the messages can be customized as `flowControlPause: string` and `flowControlResume: string` in the constructor options. `PAUSE` and `RESUME` are not passed to the underlying pseudoterminal if flow control is enabled.

## Troubleshooting

### Powershell gives error 8009001d

> Internal Windows PowerShell error.  Loading managed Windows PowerShell failed with error 8009001d.

This happens when PowerShell is launched with no `SystemRoot` environment variable present.

### ConnectNamedPipe failed: Windows error 232

This error can occur due to anti-virus software intercepting winpty from creating a pty. To workaround this you can exclude this file from your anti-virus scanning `node-pty\build\Release\winpty-agent.exe`

## pty.js

This project is forked from [chjj/pty.js](https://github.com/chjj/pty.js) with the primary goals being to provide better support for later Node.js versions and Windows.

## License

* Copyright (c) 2012-2015, Christopher Jeffrey (MIT License).
* Copyright (c) 2016, Daniel Imms (MIT License).
* Copyright (c) 2018, Microsoft Corporation (MIT License).
* Copyright (c) 2018, David Wilson (MIT License).
* Copyright (c) 2018, oznu (MIT License).
* Copyright (c) 2023, Homebridge (MIT License).
