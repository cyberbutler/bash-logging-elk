# Docker Bash Logging ELK Stack
This project uses [Anthony Lapenna’s Docker Compose repository for ELK](https://github.com/deviantony/docker-elk) as its base. The `operator` container was built by @cyberbutler as well as the Logstash pipeline. 

You can read the full article on how this repository works [here](https://medium.com/maverislabs/logging-bash-history-cefdce602595?sk=f5b972dc7bf99c2409add045c388a483). 

## Start the stack
```bash
docker-compose up --build -d
```

Launch a bash shell in the `operator` container:
```bash
docker exec -it bash-logging-elk_operator_1 bash
```

## Deploy with Ansible
If you would rather deploy the bash logging configuration to existing or new infrastructure as opposed to manually replicating the configuration I've described in the bash-operator container, you can use the ansible playbook included in this repository.
```bash
cd ansible/
cp inventory.yml.example inventory.yml
# Modify inventory.yml
ansible-playbook playbooks/configure_bash_logging.yml
```

## How to use
All commands are logged to `/var/log/bash.log` using `rsyslog`. `filebeat` pushes those logs to the `logstash` container over TCP 5044. By default you can login to Kibana at `http://localhost:5601` with the credentials `elastic:changme`.

To log the output of your commands you can either feed STDOUT to STDIN by piping your commands to `logoutput` or by using `logoutput` directly against a file:

STD Redirection:
```bash
echo "This will be logged!" | logoutput
```

File Read:
```bash
logoutput ./test.txt
```
