# PHP Debug Adapter for Visual Studio Code

[![vs marketplace](https://img.shields.io/vscode-marketplace/v/robberphex.php-debug.svg?label=vs%20marketplace)](https://marketplace.visualstudio.com/items?itemName=robberphex.php-debug)
[![downloads](https://img.shields.io/vscode-marketplace/d/robberphex.php-debug.svg)](https://marketplace.visualstudio.com/items?itemName=robberphex.php-debug)
[![rating](https://img.shields.io/vscode-marketplace/r/robberphex.php-debug.svg)](https://marketplace.visualstudio.com/items?itemName=robberphex.php-debug)
[![build](https://img.shields.io/travis/robberphex/vscode-php-debug/master.svg?label=build)](https://travis-ci.org/robberphex/vscode-php-debug)
[![codecov](https://codecov.io/gh/robberphex/vscode-php-debug/branch/master/graph/badge.svg)](https://codecov.io/gh/robberphex/vscode-php-debug)
[![code style: prettier](https://img.shields.io/badge/code_style-prettier-ff69b4.svg)](https://github.com/prettier/prettier)

![Demo GIF](images/demo.gif)

## Installation

Install the extension: Press `F1`, type `ext install php-debug`.

This extension is a debug adapter between VS Code and [Xdebug](https://xdebug.org/) by Derick Rethan. Xdebug is a PHP extension (a `.so` file on Linux and a `.dll` on Windows) that needs to be installed on your server.

1. [Install Xdebug](https://xdebug.org/docs/install)

2. [Configure PHP to use Xdebug](https://xdebug.org/docs/install#configure-php) by adding `zend_extension=path/to/xdebug` to your php.ini. The path of your php.ini is shown in your `phpinfo()` output under "Loaded Configuration File".

3. Enable remote debugging in your `php.ini`:

   ```ini
   [Xdebug]
   xdebug.remote_enable = 1
   xdebug.remote_autostart = 1
   ```

   There are other ways to tell Xdebug to connect to a remote debugger than `remote_autostart`, like cookies, query parameters or browser extensions. I recommend `remote_autostart` because it "just works". There are also a variety of other options, like the port (by default 9000), please see the [Xdebug documentation on remote debugging](https://xdebug.org/docs/remote#starting) for more information.

4. If you are doing web development, don't forget to restart your webserver to reload the settings.

5. Verify your installation by checking your `phpinfo()` output for an Xdebug section.

### VS Code Configuration

In your project, go to the debugger and hit the little gear icon and choose _PHP_. A new launch configuration will be created for you with two configurations:

- **Listen for Xdebug**
  This setting will simply start listening on the specified port (by default 9000) for Xdebug. If you configured Xdebug like recommended above, everytime you make a request with a browser to your webserver or launch a CLI script Xdebug will connect and you can stop on breakpoints, exceptions etc.
- **Launch currently open script**
  This setting is an example of CLI debugging. It will launch the currently opened script as a CLI, show all stdout/stderr output in the debug console and end the debug session once the script exits.

#### Supported launch.json settings:

- `request`: Always `"launch"`
- `hostname`: The address to bind to when listening for Xdebug (default: all IPv6 connections if available, else all IPv4 connections)
- `port`: The port on which to listen for Xdebug (default: `0`)
- `stopOnEntry`: Wether to break at the beginning of the script (default: `false`)
- `pathMappings`: A list of server paths mapping to the local source paths on your machine, see "Remote Host Debugging" below
- `log`: Wether to log all communication between VS Code and the adapter to the debug console. See _Troubleshooting_ further down.
- `ignore`: An optional array of glob patterns that errors should be ignored from (for example `**/vendor/**/*.php`)
- `xdebugSettings`: Allows you to override Xdebug's remote debugging settings to fine tuning Xdebug to your needs. For example, you can play with `max_children` and `max_depth` to change the max number of array and object children that are retrieved and the max depth in structures like arrays and objects. This can speed up the debugger on slow machines.
  For a full list of feature names that can be set please refer to the [Xdebug documentation](https://xdebug.org/docs-dbgp.php#feature-names).
  - `max_children`: max number of array or object children to initially retrieve
  - `max_data`: max amount of variable data to initially retrieve.
  - `max_depth`: maximum depth that the debugger engine may return when sending arrays, hashs or object structures to the IDE.
  - `show_hidden`: This feature can get set by the IDE if it wants to have more detailed internal information on properties (eg. private members of classes, etc.) Zero means that hidden members are not shown to the IDE.

Options specific to CLI debugging:

- `program`: Path to the script that should be launched
- `args`: Arguments passed to the script
- `cwd`: The current working directory to use when launching the script
- `runtimeExecutable`: Path to the PHP binary used for launching the script. By default the one on the PATH.
- `runtimeArgs`: Additional arguments to pass to the PHP binary
- `externalConsole`: Launches the script in an external console window instead of the debug console (default: `false`)
- `env`: Environment variables to pass to the script

## Features

- Line breakpoints
- Conditional breakpoints
- Function breakpoints
- Step over, step in, step out
- Break on entry
- Breaking on uncaught exceptions and errors / warnings / notices
- Multiple, parallel requests
- Stack traces, scope variables, superglobals, user defined constants
- Arrays & objects (including classname, private and static properties)
- Debug console
- Watches
- Run as CLI
- Run without debugging

## Remote Host Debugging

To debug a running application on a remote host, you need to tell Xdebug to connect to a different IP than `localhost`. This can either be done by setting [`xdebug.remote_host`](https://xdebug.org/docs/remote#remote_host) to your IP or by setting [`xdebug.remote_connect_back = 1`](https://xdebug.org/docs/remote#remote_connect_back) to make Xdebug always connect back to the machine who did the web request. The latter is the only setting that supports multiple users debugging the same server and "just works" for web projects. Again, please see the [Xdebug documentation](https://xdebug.org/docs/remote#communcation) on the subject for more information.

To make VS Code map the files on the server to the right files on your local machine, you have to set the `pathMappings` settings in your launch.json. Example:

```json
// server -> local
"pathMappings": {
  "/var/www/html": "${workspaceRoot}/www",
  "/app": "${workspaceRoot}/app"
}
```

Please also note that setting any of the CLI debugging options will not work with remote host debugging, because the script is always launched locally. If you want to debug a CLI script on a remote host, you need to launch it manually from the command line.

## Troubleshooting

- If you think you found a bug, [open an issue](https://github.com/felixfbecker/vscode-php-debug/issues)
- Make sure you have the latest version of this extension and Xdebug installed
- Try out a simple PHP file to recreate the issue, for example from the [testproject](./testproject)
- In your php.ini, set [`xdebug.remote_log = /path/to/logfile`](https://xdebug.org/docs/remote#remote_log)
  (make sure your webserver has write permissions to the file)
- Set `"log": true` in your launch.json

## Contributing

To hack on this adapter, clone the repository and open it in VS Code. You need NodeJS and typings installed (`npm install -g typings`). Install dependencies by running `npm install` and `typings install`.

You can debug the extension (run it in "server mode") by selecting the "Debug adapter" launch configuration and hitting `F5`. Then, open a terminal inside the project, and open the included testproject with VS Code while specifying the current directory as `extensionDevelopmentPath`:

```sh
code testproject --extensionDevelopmentPath=.
```

VS Code will open an "Extension Development Host" with the debug adapter running. Open `.vscode/launch.json` and uncomment the `debugServer` configuration line. Hit `F5` to start a debugging session. Now you can debug the testproject like specified above and set breakpoints inside your first VS Code instance to step through the adapter code.

The extension is written in TypeScript and compiled using a Gulpfile that first transpiles to ES6 and then uses Babel to specifically target VS Code's Node version. You can run the compile task through `npm run compile`, `gulp compile` or from VS Code with `Ctrl`+`Shift`+`B`. `npm run watch` / `gulp watch` enables incremental compilation.

Tests are written with Mocha and can be run with `npm test`. The tests are run in CI on Linux and Windows against PHP 7.0 ~ 7.3 and Xdebug [3.0.0](https://github.com/RobberPhex/xdebug/releases/tag/3.0.0).
