## tmux
Sessions
 -> Windows/Screens/Tabs
  -> Splits/Panes

PREFIX is `CTRL,b`

|Shortcut|Description|
-|-
tmux new -s SESSIONNAME | 
tmux ls | Lists sessions
PREFIX,c | Create new session
PREFIX,w | Show all sessions, windows, and tabs. Arrows to browse; enter to select
PREFIX,% | Vertical split
PREFIX," | Horizontal split

```
$ cat ~/.inputrc
"\e[A":history-search-backward
"\e[B":history-search-forward
"\e[1;5C": forward-word
"\e[1;5D": backward-word
set completion-ignore-case On
```
```
$ cat ~/.bashrc
export HISTTIMEFORMAT="%Y-%m-%d %T # "
alias rev="bash.exe rev.sh"
$ source ~/.bashrc
```

## Gnome
In terminal
```
gsettings set org.gnome.desktop.wm.preferences resize-with-right-button true
gsettings set org.gnome.desktop.wm.preferences mouse-button-modifier '<Alt>'
```

```
$ echo $PS1
\[\033]0;$TITLEPREFIX:${PWD//[^[:ascii:]]/?}\007\]\n\[\033[32m\]\u@\h \[\033[35m\]$MSYSTEM \[\033[33m\]\w\[\033[0m\]\n$
export PS1="\[\033[33m\]\w\[\033[0m\]\n# "
```

```
$ cat file.csv | head -n1 | sed 's/,/\n/g' | nl | tail -n1 | sed -r 's/([0-9]+).*/\1/g' | xargs
$ cat file.csv | cut -d "," -f 62 | rev | sort | rev
```

## Cut after number of chars:
```
$ cut -c-10
$ cut -c91- *v03.log | nl
```

## Cut after a mtaching word:
```
$ head -n3 2017-09-29\ stalled.csv | grep -oP '(?<=MsgSourceID = ).*' | nl
```

## Docker
Install https://download.docker.com/win/stable/DockerToolbox.exe, open it an run
```
docker pull kalilinux/kali-linux-docker
docker run -it <imageID> /bin/bash
```

Generate password
```
openssl rand -base64 32
```

## .NET Core Linux
```
> dotnet --version
```
```
mkdir testapp
cd testapp
dotnet new 
dotnet restore
dotnet run
```
