# aliases
Zsh/Bash aliases/function aliases that is useful for Docker. Simply add these to $HOME/.zprofile.

## Alias List (See descriptions below):
### Highlighted ones are extremely recommended
- zshrc
- zprofile
- **treedir**
- **dcedit**
- dcdown
- **dcup**
- **dclogs**
- dcrestart
- dcupgrade
- dbuild
- dlogs
- **free_ports**
```bash
alias homelab-cfg='sudo nano /etc/nginx/sites-available/homelab && sudo nginx -t && sudo systemctl restart nginx'
# edit and save .zshrc
zshrc() {
  nano $HOME/.zshrc
  source $HOME/.zshrc
}
# edit and save .zprofile
zprofile() {
  nano $HOME/.zprofile
  source $HOME/.zprofile
}
# treedir <dir>
# list total size of all sub-directories under <dir>
function treedir {
  sudo du -sh $1/* | sort -hr;
}

DOCKER_COMPOSE_FILES=('compose.yml' 'compose.yaml' 'docker-compose.yml' 'docker-compose.yaml')
# dcedit (under stack directory)
# create default .gitignore if not exists
# find existing compose yaml file
# nano compose.yml if not exists
# call `dcup` if file content was edited
function dcedit {
  if [ ! -f '.gitignore' ]; then
    echo -e '*\n!compose.yml' > .gitignore
    echo added .gitignore
    cat .gitignore
  fi
  found=0
  for file in ${DOCKER_COMPOSE_FILES[@]};
  do
    if [ -f $file ]; then
      found=1
      break
    fi
  done
  if [[ $found == 0 ]]; then
    echo 'Creating compose.yml'
    unset file
    file='compose.yml'
    touch $file
  fi
  oldtime=`stat -c %Y "$file"`
  nano $file
  # if file changed
  if [[ `stat -c %Y "$file"` -gt $oldtime ]]; then
	docker compose up -d --remove-orphans
  else
    echo file unchanged
  fi
}
# dcdown (under stack directory)
alias dcdown='docker compose down --remove-orphans'
# dcup (under stack directory)
alias dcup='docker compose up -d --remove-orphans'
# dclogs (under stack directory)
alias dclogs='docker compose logs -f --tail 10'
# dcrestart (under stack directory)
alias dcrestart='dcdown -t 1 && dcup'
# dbuild (under Dockerfile directory, uses the folder name as image name)
alias dbuild='docker build . -t $(basename $PWD)'
# dcupgrade (under stack directory)
dcupgrade() {
    docker compose pull
    docker compose up -d
}
# dlogs <container_name>
alias dlogs='docker logs -f --tail 10'
# free_ports <starting_port> <limit>
free_ports() {
    echo "
import socket
max=$2
count=0
for p in range($1, 65535):
    if count >= max:
        break
    try:
        s=socket.socket()
        s.bind(('', p))
        s.close()
        print(p)
        count+=1
    except:
        pass
" | python3
}
```
