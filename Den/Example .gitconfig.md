```bash
[user]
        name = Wis Tomasz
        email = tomaszrwis@gmail.com
[push]
        default = simple
[core]
        editor = nvim
        longpaths = true
[alias]
  ci = commit
  co = checkout
  s = status
  st = status -sb
  lo = log --format='%Cred%h%Creset %s %Cgreen(%cr) %C(blue)<%an>%Creset%C(yellow)%d%Creset' --no-merges
  lon = !git --no-pager log --format='%Cred%h%Creset %s %Cgreen(%cr) %C(blue)<%an>%Creset%C(yellow)%d%Creset' --no-merge
s
  lg = log --graph --pretty=format:'%Cred%h%Creset -%C(yellow)%d%Creset %s %Cgreen(%cr) %C(bold blue)<%an>%Creset' --abb
rev-commit
  lgn = !git --no-pager log --graph --pretty=format:'%Cred%h%Creset -%C(yellow)%d%Creset %s %Cgreen(%cr) %C(bold blue)<%
an>%Creset' --abbrev-commit
  lg1 = !git lon -1
  synchronizeModules = !git submodule sync --recursive && git submodule update --init --recursive
  br = branch -a --no-pager
  bl = branch --no-pager
```

#arch #linux #devops #setup #config