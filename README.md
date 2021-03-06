# sudo-prompt

Run a non-graphical terminal command using `sudo`, prompting the user with a graphical OS dialog if necessary. Useful for background Node.js applications or native Electron apps that need sudo.

![A sudo prompt on OS X for an app called "Ronomon"](./osx.png)

`sudo-prompt` provides a native OS dialog prompt on **OS X** and **Linux** with custom name and optional icon.

`sudo-prompt` has no external dependencies and does not require any native bindings.

## Installation
```
npm install sudo-prompt
```

## Usage
Note: Your command should not start with the `sudo` prefix.
```
var sudo = require('sudo-prompt');
var options = {
  name: 'Ronomon',
  icns: '/path/to/icns/file', // (optional)
};
sudo.exec('echo hello', options, function(error, stdout, stderr) {});
```

`sudo-prompt` will use `process.title` as `options.name` if `options.name` is not provided. `options.name` must be alphanumeric only (spaces are supported) and at most 70 characters.

Your command should not depend on any current working directory or environment variables in order to execute correctly, and you should take care to use absolute paths and not relative paths.

## Behavior
On OS X, `sudo-prompt` should behave just like the `sudo` command in the shell. If your command does not work with the `sudo` command in the shell (perhaps because it uses `>` redirection to a restricted file), then it may not work with `sudo-prompt`. However, it is still possible to use sudo-prompt to get a privileged shell, [see this closed issue for more information](https://github.com/jorangreef/sudo-prompt/issues/1).

On Linux, `sudo-prompt` will use either `gksudo`, `pkexec`, or `kdesudo` to show the password prompt and run your command. Where possible, `sudo-prompt` will try and get these to mimic `sudo`. Depending on which binary is used, and due to the limitations of some binaries, the name of your program or the command itself may be displayed to your user. Passing `options.icns` is currently not supported by `sudo-prompt` on Linux. Patches are welcome to add support for icons based on `polkit`.

## Non-graphical terminal commands only
Just as you should never use `sudo` to launch any graphical applications, you should never use `sudo-prompt` to launch any graphical applications. Doing so could cause files in your home directory to become owned by root. `sudo-prompt` is explicitly designed to launch non-graphical terminal commands. For more information, [read this post](http://www.psychocats.net/ubuntu/graphicalsudo).

## Concurrency
On systems where the user has opted to have `tty-tickets` enabled, each call to `exec()` will result in a separate password prompt. Where `tty-tickets` are disabled, subsequent calls to `exec()` (but not concurrent calls) will not require a password prompt, so long as the user's `sudo` timestamp file remains valid.

You should never rely on `sudo-prompt` to execute your calls in order. If you need to enforce ordering of calls, then you should explicitly order your calls in your application. Where your commands are short-lived, you should queue your calls to `exec()` to make sure your user is not overloaded with password prompts.

## Invalidating the timestamp
You can invalidate the user's `sudo` timestamp file to force the prompt to appear by running the following command in your terminal:

```sh
$ sudo -k
```
