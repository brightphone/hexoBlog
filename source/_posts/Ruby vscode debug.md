---
layout: post
title: "Ruby VSCode debug 环境配置"
date: 2019-07-20 12:00:00
comments: true
catagories: language
tags: [Ruby]
---

# 安装需要的gems
```
gem install ruby-debug-ide
gem install debase
gem install solargraph
gem install rubocop
gem install rcodetools
gem install reek
```
安装这些gem，如果提示没有权限就加上sudo。
可以[参考博客](https://blog.csdn.net/skybboy/article/details/80105524)

去VSCode的Extensions市场搜索Ruby,安装Ruby，Ruby Solargraph,VSCode Ruby这几个插件
去VSCode debug创建lauch.json文件
![image](/res/images/article/rubyvscode/1.png)
![image](/res/images/article/rubyvscode/2.png)
![image](/res/images/article/rubyvscode/3.png)
lauch.json默认配置如下
```
{
    // Use IntelliSense to learn about possible attributes.
    // Hover to view descriptions of existing attributes.
    // For more information, visit: https://go.microsoft.com/fwlink/?linkid=830387
    "version": "0.2.0",
    "configurations": [
        {
            "name": "Debug Local File",
            "type": "Ruby",
            "request": "launch",
            "program": "${workspaceRoot}/main.rb"
        }
    ]
}
```
可配置项有下面这些，可以去官网看看每个的含义[官方说明](https://github.com/rubyide/vscode-ruby/wiki/2.-Launching-from-VS-Code#available-vs-code-defined-variables)

# 配置launch.json文件

## "name"
You can set this to what ever you want. It will need to be unique within the configurations array. This is the string that will be shown in the drop-down selector on the debug side bar.

## "type"
Must be "Ruby". This tells VS Code what debugger to run.

## "request"
Either "launch" - which enables launching the provided program directly from VS Code - or "attach" - which allows you to attach to a remote debug session.

## "program"
This is the ruby script that will first be launched when debugging is started. You should not rely on relative paths working. If the file is in your workspace (which is usually is) this string should have the structure "${workspaceRoot}/path/to/script.rb".

You could debug the current open file with just "program": "${file}".

## "cwd"

By default, the working directory is set to the location of the file provided in the program string. It is common for this value to be set to "cwd": "${workspaceRoot}".

## "stopOnEntry"
Stop program execution on the first line always. Note that all active breakpoints are set before the debugger starts even if this field is not set. Valid options are true and false (default).
(这个我目前设置了，貌似没有起作用)

## "showDebuggerOutput"
Provide some extra output to the debug terminal, specifically about the running of rdebug-ide. Valid options are true and false (default).

## "args"

An array of arguments to provide to the script under debug. Each string in the array is sent as a seperate argument, so if you would call the script from a terminal as:
```
ruby main.js --infile '/the/file name/with spaces' --count 3 --base
```
this setting should read:
```
	"args": ["--infile", "/the/file name/with spaces", "--count", "3"]
```
There is no need to include the quotes around the second argument. Doing so would actually result in the string WITH the quotes being passed to the script.

## "env"

Provide a hash of environment variable to set before launching the program. For example:

```
"env": {
		"BASE": "${workspaceRoot}",
		"EXT": "${fileExtname}",
		"RAILS_ENV": "test"
	}
```

## "pathToRDebugIDE":
Set the absolute path to rdebug-ide if it's not in your PATH. On windows you need to provide the .bat file.

## "useBundler":
Run rdebug-ide within bundler exec. You may need to do this if you've installed the ruby-debug-ide gem within your project. Valid options are true and false (default).

## "pathToBundler":
If you have useBundler set, and bundler isn't in your PATH, set the absolute path here. If you're wrapping in bundler, you shouldn't need to provide a pathToRDebugIDE setting. On windows you need to provide the .bat file.

Also, if you've installed with the bundler install --binstubs you should be able to skip the bundler related settings and use "pathToRDebugIDE": "${workspaceRoot}/bin/rdebug-ide".

## Available VS Code defined variables
下面是VSCode自定义的一些变量代表的信息
```
${workspaceRoot}:the path of the folder opened in VS Code
${file}	:the current opened file
${fileBasename}:the current opened file's basename
${fileDirname}:	the current opened file's dirname
${fileExtname}:	the current opened file's extension
```

## 远程调试
远程调试需要设定
```
{
	"version": "0.2.0",
	"configurations": [
		{
			"name": "Debug Attach",
			"type": "Ruby",
			"request": "attach",
			"cwd": "${workspaceRoot}",
			"remoteWorkspaceRoot": "${workspaceRoot}",
			"remoteHost": "SERVER_IP_ADDRESS",
			"remotePort": "SERVER_LISTENING_PORT"
		}
	]
}
```
## "remoteWorkspaceRoot"
Set this to the remote base path. The cwd setting above will be dropped from the start of any paths passed to the debugger and this value will replace it. This enables different paths to be used between the development and test devices.

With:
```
"cwd": "/dev/folder",
"remoteWorkspaceRoot": "/usr/local/www"
```
The path: /dev/folder/app/models/user.rb will become /usr/local/www/app/models/user.rb. Without this translation, break points won't be set correctly.

## "remoteHost"
The IP address of the server the debugger will be running on. If you're running it locally, use 127.0.0.1 rather than localhost.

## "remotePort"
The port to connect to on the remote server. The default listening port for ruby-debug-ide is 1234.
 ## Running the debugger
 Run the debugger from the command line with:

```
rdebug-ide --host SERVER_IP_ADDRESS\
    --port SERVER_LISTENING_PORT\
    -- TARGET_SCRIPT {...script args} 
```

Where the SERVER_IP_ADDRESS and SERVER_LISTENING_PORT match the settings in your launch.json configuration. If there is only one network device on the server where the debugger is running, (and/or you have considered the security implications) you can use 0.0.0.0 to allow the debugger to listen on any connection.
```
{
    "diffEditor.ignoreTrimWhitespace": false,
    "editor.wordWrap": "on",
    "window.zoomLevel": 0,
    "editor.minimap.enabled": false,
    "workbench.colorTheme": "Xcode Default (Dark)",
    "[markdown]": {},
    "[ruby]": {},
    "files.associations": {
        "*.rb": "ruby"
    },
    "ruby.lint": {
        "rubocop": {
            "useBundler": false // enable rubocop via bundler
        },
        "reek": {
            "useBundler": false // enable reek via bundler
        }
    },
    "ruby.codeCompletion": "rcodetools",
    "ruby.format": "rubocop", // use rubocop for formatting
    "ruby.intellisense": "rubyLocate",
    "ruby.useLanguageServer": true, // use the internal language server (see below)
    "ruby.useBundler": false, //run non-lint commands with bundle exec
    "ruby.interpreter.commandPath": "ruby",
    "ruby.locate": {
        "exclude": "{**/@(test|spec|tmp|.*),**/@(test|spec|tmp|.*)/**,**/*_spec.rb}",
        "include": "**/*.rb"
    }
}
```