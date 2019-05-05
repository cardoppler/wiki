### Install Git for VSCode on Windows
1. Install https://git-scm.com/download/win
- During setup wizard select `Use Visual Studio Code as Git's default editor`
- Select `Command line and also 3rd party`
- `OpenSSL`
- `Windows-style`
- `MinTTY`
- Check: `system caching`, `cred manager`

2. Open VSCode, `F1` > `Git: Clone`

### Shortcuts
Shortcut | Description
:-:| - 
CTRL + K, CTRL + O | Open directory 
CTRL + K, V | Markdown preview 
CTRL + ALT + UP/DOWN | Multiple cursors
ALT + Click | Multiple cursors
SHIFT + ALT + I | Cursor at the end of line
SHIFT + CTRL + L | Cursor on same words
CTRL + B | Toggle side bar
SHIFT + ALT + 0 | Toggle horizontal/vertical split
ALT + UP/DOWN | Move line
CTRL + K, Z | Zen mode
SHIFT + ALT + F | Format
SHIFT + ALT + click&drag | Vertical selection with mouse

# Trips & Tricks
Added “- “ (dash space) in front of every line that started with a lower case using $1 in the **replace field** to reference the unnamed group from the **search field**

Before:

![Image](https://cardoppler.tech/static/vscode_02.png)

After replacement:

![Image](https://cardoppler.tech/static/vscode_01.png)

### Extensions
- bajdzis.vscode-database-1.2.0
- donjayamanne.githistory-0.2.1
- donjayamanne.python-0.6.7
- DotJoshJohnson.xml-1.9.2
- humao.rest-client-0.14.6
- jmrog.vscode-nuget-package-manager-1.1.4
- ms-mssql.mssql-1.0.0
- ms-vscode.csharp-1.11.0
- ms-vscode.powershell-1.4.1
- swyphcosmo.spellchecker-1.2.13
- Tyriar.sort-lines-1.3.0

### settings.json
```
{
    "git.enableSmartCommit": true,    
    "spellchecker.language": "en_GB-ise",
    // "terminal.integrated.shell.windows": "C:\\Windows\\System32\\WindowsPowerShell\\v1.0\\powershell.exe",
    "terminal.integrated.shell.windows": "C:\\Users\\pietriri\\AppData\\Local\\Programs\\Git\\usr\\bin\\bash.exe",
    "terminal.integrated.shellArgs.windows": ["--login","-i"],
    "window.reopenFolders": "all",
    "workbench.activityBar.visible": false,
    "workbench.colorTheme": "Visual Studio Light",
    "workbench.editor.showTabs": false,
    "workbench.iconTheme": "vscode-icons",
    // "http.proxy": "http://username:PASSWORD@someproxy.com:8080",
    // "http.proxyStrictSSL": false
    "workbench.welcome.enabled": false,
    "editor.fontSize": 15,
    "workbench.sideBar.location": "left",    
    "files.exclude": {
        "**/*.png": true,
        "**/*.PNG": true,
        "**/*.db": true,
        "**/*.pdf": true,
        "**/*.msg": true,
        "**/*.docx": true,
        "**/*.xlsx": true,
        "**/*.rdp": true,
        "**/*.ini": true,
        "$RECYCLE.BIN/" : true,
        "Add-in Express/" : true,
        "My Received Files/" : true,
        "My Web Sites/" : true,
        "Visual Studio 2010/" : true,
        "Visual Studio 2015/" : true
    },
    "files.associations": {
        "*.http": "http"
    },
    "terminal.integrated.cursorStyle": "block",
    "editor.renderWhitespace": "all",
    "window.zoomLevel": -2
}
```

# keybindings.json
```
// Place your key bindings in this file to overwrite the defaults
[{
    "key": "ctrl+left",
    "command": "cursorWordEndLeft",
    "when": "editorTextFocus"
},{
    "key": "ctrl+right",
    "command": "cursorWordStartRight",
    "when": "editorTextFocus"
},
    {
        "key": "ctrl+shift+enter",
        "command": "workbench.action.toggleMaximizedPanel"
    },
    {
        "key": "f8",
        "command": "workbench.action.terminal.runSelectedText"
    },
        {
        "key": "ctrl+shift+b",
        "command": "workbench.action.toggleSidebarVisibility"
    },
    {
        "key": "ctrl+b",
        "command": "editor.action.insertSnippet",
        "args": { "snippet": "**$TM_SELECTED_TEXT**" }
    }
]
```
