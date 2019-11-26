# Linux



### Let's Encrypt / certbot

Example using DigitalOcean API to perfom TXT DNS verification \(instead of website verification\) Get the API key form Digital Ocean admin page

* [https://certbot-dns-digitalocean.readthedocs.io/en/stable/](https://certbot-dns-digitalocean.readthedocs.io/en/stable/) 

**Installation**

```text
apt install software-properties-common
sudo add-apt-repository universe
sudo add-apt-repository ppa:certbot/certbot
apt update
apt install certbot
apt install python3-certbot-dns-digitalocean
```

Then go to DigitalOcean Admin page &gt; API &gt; Tokens/Keys &gt; Generate New Personal Access Token and paste the token inot a file with the following format:

```text
cat digital_ocean_API_creds_used_by_certbot
dns_digitalocean_token = 8c...f9
```

**Usage**

```text
certbot --help

certbot certonly  --register-unsafely-without-email --dns-digitalocean --dns-digitalocean-credentials ~/certbot/digital_ocean_API_creds_used_by_certbot -d basedomain.com -d www.basedomain.com -d anothersubdomainfor.basedomain.com

certbot certificates # list existing certificates
```

### tmux

Sessions -&gt; Windows/Screens/Tabs -&gt; Splits/Panes

Thedefault PREFIX is `CTRL,b`; to change to `CTRL,a`:

```text
vim ~/.tmux.conf
# remap prefix from 'C-b' to 'C-a'
unbind C-b
set-option -g prefix C-a
bind-key C-a send-prefix
```

Then reload config with:

```text
PREFIX : source-file ~/.tmux.conf
```

| Shortcut | Description |  |
| :--- | :--- | :--- |
| tmux ls | Lists sessions |  |
| tmux new -s SESSIONNAME | Create new session |  |
| tmux a -t SESSIONNAME | Attach to session |  |
| PREFIX c | Create new session |  |
| PREFIX w | Show all sessions, windows, and tabs. Arrows to browse; enter to select |  |
| PREFIX % | Vertical split |  |
| PREFIX " | Horizontal split |  |
| PREFIX | : source-file ~/.tmux.conf | Reload tmux config |
| PREFIX , | Rename Pane |  |
| PREFIX Z | Zoom pane |  |

#### inputrc

```text
$ vim ~/.inputrc
"\e[A":history-search-backward
"\e[B":history-search-forward
"\e[1;5C": forward-word
"\e[1;5D": backward-word
set completion-ignore-case On
```

#### bashrc

```text
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

### Gnome

In terminal

```text
gsettings set org.gnome.desktop.wm.preferences resize-with-right-button true
gsettings set org.gnome.desktop.wm.preferences mouse-button-modifier '<Alt>'
```

```text
$ echo $PS1
\[\033]0;$TITLEPREFIX:${PWD//[^[:ascii:]]/?}\007\]\n\[\033[32m\]\u@\h \[\033[35m\]$MSYSTEM \[\033[33m\]\w\[\033[0m\]\n$
export PS1="\[\033[33m\]\w\[\033[0m\]\n# "
```

```text
$ cat file.csv | head -n1 | sed 's/,/\n/g' | nl | tail -n1 | sed -r 's/([0-9]+).*/\1/g' | xargs
$ cat file.csv | cut -d "," -f 62 | rev | sort | rev
```

### Cut after number of chars:

```text
$ cut -c-10
$ cut -c91- *v03.log | nl
```

### Cut after a mtaching word:

```text
$ head -n3 2017-09-29\ stalled.csv | grep -oP '(?<=MsgSourceID = ).*' | nl
```

### tr

```text
tr -d "\n" # removes a newline character
tr -- '+=/' '-_~' #  replaces characters that are not valid o the left with characters that are valid on the right.
```

### Docker

Install [https://download.docker.com/win/stable/DockerToolbox.exe](https://download.docker.com/win/stable/DockerToolbox.exe), open it an run

```text
docker pull kalilinux/kali-linux-docker
docker run -it <imageID> /bin/bash
```

Generate password

```text
openssl rand -base64 32
```

### .NET Core Linux

```text
> dotnet --version
```

```text
mkdir testapp
cd testapp
dotnet new 
dotnet restore
dotnet run
```

