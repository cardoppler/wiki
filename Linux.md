## Let's Encrypt / certbot
Example using DigitalOcean API to perfom TXT DNS verification (instead of website verification)
Get the API key form Digital Ocean admin page
- https://certbot-dns-digitalocean.readthedocs.io/en/stable/ 

#### Installation
```
apt install software-properties-common
sudo add-apt-repository universe
sudo add-apt-repository ppa:certbot/certbot
apt update
apt install certbot
apt install python3-certbot-dns-digitalocean
```

Then go to DigitalOcean Admin page > API > Tokens/Keys > Generate New Personal Access Token
and paste the token inot a file with the following format:
```
cat digital_ocean_API_creds_used_by_certbot
dns_digitalocean_token = 8c...f9
```

#### Usage
```
certbot --help

certbot certonly  --register-unsafely-without-email --dns-digitalocean --dns-digitalocean-credentials ~/certbot/digital_ocean_API_creds_used_by_certbot -d basedomain.com -d www.basedomain.com -d anothersubdomainfor.basedomain.com

certbot certificates # list existing certificates
```

## tmux
Sessions
 -> Windows/Screens/Tabs
  -> Splits/Panes

The default  PREFIX is `CTRL,b`; to change to `CTRL,a`:
```
vim ~/.tmux.conf
# remap prefix from 'C-b' to 'C-a'
unbind C-b
set-option -g prefix C-a
bind-key C-a send-prefix

#copy-paste vi-like
set-window-option -g mode-keys vi
bind P paste-buffer
bind-key -T copy-mode-vi v send-keys -X begin-selection
bind-key -T copy-mode-vi y send-keys -X copy-selection
bind-key -T copy-mode-vi r send-keys -X rectangle-toggle

# split panes using | and -
bind | split-window -h
bind - split-window -v
unbind '"'
unbind %

# switch panes using Alt-arrow without prefix
bind -n M-Left select-pane -L
bind -n M-Right select-pane -R
bind -n M-Up select-pane -U
bind -n M-Down select-pane -D

# Enable mouse control (clickable windows, panes, resizable panes)
set -g mouse on

# don't rename windows automatically
set-option -g allow-rename off
```
Then reload config with: 
```
PREFIX : source-file ~/.tmux.conf
```

Shortcut|Description|
-|-
tmux ls | Lists sessions
tmux new -s SESSIONNAME | Create new session
tmux a -t SESSIONNAME | Attach to session
PREFIX c | Create new session
PREFIX w | Show all sessions, windows, and tabs. Arrows to browse; enter to select
PREFIX \| | Vertical split
PREFIX - | Horizontal split
PREFIX | : source-file ~/.tmux.conf | Reload tmux config
PREFIX , | Rename Pane
PREFIX Z | Zoom pane
ALT + arrow | Move

### Copy-paste
Shortcut|Description
-|-
PREFIX [ | to go into copy mode
 | move to start
v | to start copying
 | Select text to be copied
y |  to copy into Tmux buffer
PREFIX ] | to paste

## inputrc
```
$ vim ~/.inputrc
"\e[A":history-search-backward
"\e[B":history-search-forward
"\e[1;5C": forward-word
"\e[1;5D": backward-word
set completion-ignore-case On
```

### bashrc
```
$ vim ~/.bashrc
# Time format "yyyy-mm-dd hh:mm"
export HISTTIMEFORMAT="%Y-%m-%d %T # "
# Avoid duplicates
export HISTCONTROL=ignoredups:erasedups
# When the shell exits, append to the history file instead of overwriting it
shopt -s histappend
# After each command, append to the history file and reread it
export PROMPT_COMMAND="${PROMPT_COMMAND:+$PROMPT_COMMAND$'\n'}history -a; history -c; history -r"
# Big history
export HISTSIZE=100000
export HISTFILESIZE=100000

$ source .bashrc
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

## tr
```
tr -d "\n" # removes a newline character
tr -- '+=/' '-_~' #  replaces characters that are not valid o the left with characters that are valid on the right.
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
