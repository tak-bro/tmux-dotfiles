# ref: [tmux-tmuxinator-setup.md](https://gist.github.com/colmarius/f8c0c824b87db9279222)

### 1. Install tmux + tmuxinator

    gem install tmuxinator

### 2. Add ~/.tmuxinator project specific configurations

```
# File: ~/.tmuxinator/project-name.yml

name: project-name
root: ~/Work/ProjectName
windows:
  - App:
      layout: tiled
      panes:
        - clear && cd App
        - clear && ./boot-app App
  - api:
      layout: tiled
      panes:
        - clear && cd api
        - clear && ./boot-app api
  - sso:
      layout: tiled
      panes:
        - clear && cd sso
        - clear && ./boot-app sso
```

### 3. Set your ~/.tmux.conf

Checkout this [simple config](https://github.com/colmarius/dot-files/blob/master/files/.tmux.conf) and this more [advanced config](https://github.com/gpakosz/.tmux/).


### 4. Add ~/bin/work script for project start/stop with tmuxinator

Latest version can be found [here](https://github.com/colmarius/dot-files/blob/master/files/bin/work) in my dot-files.

```bash
# File: ~/bin/work

#!/bin/bash

display_usage() {
  echo "This script is used to start tmuxinator project sessions."
  echo -e "\nSupposing you have a session: ~/.tmuxinator/project-name.yml"
  echo -e "\nExample usage:\n"
  echo -e "work start project-name"
  echo -e "work stop project-name \n"
}

if [ $# -ne 2 ]; then
  display_usage
  exit 1
fi

operation=$1
tmuxinator_config=$2

if [ $operation == "start" ]
  then
  tmux has-session -t $tmuxinator_config 2> /dev/null

  if [ $? != 0 ]
  then
    mux start $tmuxinator_config
  else
    echo "tmux: $tmuxinator_config session is already running."
  fi
fi

if [ $operation == "stop" ]
  then
  tmux has-session -t $tmuxinator_config 2> /dev/null

  if [ $? == 0 ]
  then
    for i in `seq 1 10`;
    do
      tmux send-keys -t $tmuxinator_config:$i.1 C-c C-m
      tmux send-keys -t $tmuxinator_config:$i.2 C-c C-m
      tmux send-keys -t $tmuxinator_config:$i.3 C-c C-m
    done

    tmux kill-session -t $tmuxinator_config
  else
    echo "tmux: no $tmuxinator_config session found."
  fi
fi
```

### 5. Usage

Start a project:

    work start project-name

Stop a project:

    work stop project-name
