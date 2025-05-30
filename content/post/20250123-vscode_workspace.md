---
title: "VSCode之workspace"
subtitle: ""
description: "单目录工作区/多目录工作区/无标题的多根目录工作区的概念和区别，以及它们各自的配置文件位置。"
date: 2025-01-03T11:53:41Z
author:      "Wayne"
image: ""
tags: ["VSCode", "workspace", "settings.json"]
categories: ["Tech"]
---

## vscode 之 workspace 概念

A Visual Studio Code workspace is the collection of one or more folders that are opened in a VS Code window (instance). {{<rawhtml>}}<span style="color:red;">In most cases, you will have a single folder opened as the workspace.</span>{{</rawhtml>}} However, depending on your development workflow, you can include more than one folder, using an advanced configuration called Multi-root workspaces.

The concept of a workspace enables VS Code to:

- Configure settings that only apply to a specific folder or folders but not others.
- Persist task and debugger launch configurations that are only valid in the context of that workspace.
- Store and restore UI state associated with that workspace (for example, the files that are opened).
- Selectively enable or disable extensions only for that workspace.

## workspace 类型和它们对应的 settings

#### Single-folder workspaces：

You don't have to do anything for a folder to become a VS Code workspace other than open the folder with VS Code.(当通过 "File > Open Folder" 打开某个目录就形成了一个 Single-folder workspaces) Once you open a folder, VS Code automatically keeps track of configuration, such as your open files or editor layout. When you reopen that folder in VS Code, the editor will be as you left it previously.

A single-folder workspace opened inside VS Code.(如下图示)  
 ![A single-folder workspace opened inside VS Code](/img/image-19.png)

{{<rawhtml>}}<strong>Single-folder workspaces 的 settings:</strong>{{</rawhtml>}} Workspace settings are stored in {{<rawhtml>}}<span style="color:red;"> .vscode/settings.json</span>{{</rawhtml>}} when you open a folder as a workspace.![Alt text](/img/image-18.png)

#### multi-root workspace 多根工作间概念

Multi-root workspaces are an advanced capability of VS Code that allows you to configure multiple distinct folders to be part of the same workspace. Instead of opening a folder as workspace(Single-folder workspaces), you open a {{<rawhtml>}}<span style="color:red;"><name>.code-workspace JSON file</span>{{</rawhtml>}} that lists all folders of the workspace. For example:

```json
{
  "folders": [
    {
      "path": "my-folder-a"
    },
    {
      "path": "my-folder-b"
    }
  ]
}
```

A multi-root workspace opened in VS Code.  
 ![alt text](/img/image-20.png)

Note: The visual difference of having a folder opened versus opening a .code-workspace file can be subtle. {{<rawhtml>}}<strong>To give you a hint that a .code-workspace file has been opened, some areas of the user interface (for example, the root of the File Explorer) show an extra (Workspace) suffix next to the name.</strong>{{</rawhtml>}}

{{<rawhtml>}}<strong>Multi-root workspace settings:</strong>{{</rawhtml>}}  
 When you open a .code-workspace as workspace, {{<rawhtml>}}<span style="color:red;">all workspace settings are added into the .code-workspace file. You can still configure settings per root folder</span>{{</rawhtml>}}, and the Settings editor will present a third setting scope called {{<rawhtml>}}<span style="color:red;">Folder Settings</span>{{</rawhtml>}}:  
 ![alt text](/img/image-21.png) {{<rawhtml>}}<span style="color:red;">Settings configured per folder override settings defined in the .code-workspace.</span>{{</rawhtml>}}

#### Untitled multi-root workspaces

Unless you have already opened a .code-workspace file, the first time you add a second folder to a workspace, VS Code automatically creates an untitled workspace. In the background, VS Code automatically maintains a untitled.code-workspace file for you that contains all the folders and workspace settings from your current session. The workspace remains untitled until you decide to save it to disk.

![alt text](/img/image-22.png)  
 An untitled multi-root workspace opened in VS Code

#### 多子目录的单根工作区和多根工作区在 EXPLORER 看起来相同但其实是不同的工作区类型

1. 注意多根工作区和单根工作区 EXPLORER 下列出的哪些目录的区别，尽管在 vscode 的文件浏览器(EXPLORER)中看起来是一样的,都有多个目录，但它们其实是不相同的概念。单根工作区的 EXPLORER 下的多个目录它们有共同的父目录(也就是通过“File > Open Folder” 打开的那个目录)；但多根工作区下面的多个目录它们是没有共同的父目录的，它们是通过打开一个 .code-workspace 文件来形成的，它们可能是位于不同的磁盘下的目录。

## References

1. [官方文档](https://code.visualstudio.com/docs/editor/workspaces)
