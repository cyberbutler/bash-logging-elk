# Docker Bash Logging ELK Stack
This project uses [Anthony Lapenna’s Docker Compose repository for ELK](https://github.com/deviantony/docker-elk) as its base. The `operator` container was built by @cyberbutler as well as the Logstash pipeline. 

You can read the full article on how this repository works [here](). 

## Start the stack
```bash
docker-compose up --build -d
```

Launch a bash shell in the `operator` container:
```bash
docker exec -it bash-logging-elk_operator_1 bash
```

## How to use
All commands are logged to `/var/log/bash.log` using `rsyslog`. `filebeat` pushes those logs to the `logstash` container over TCP 5000. By default you can login to Kibana at `http://localhost:5601` with the credentials `elastic:changme`.

To log the output of your commands you can either feed STDOUT to STDIN by piping your commands to `logoutput` or by using `logoutput` directly against a file:

STD Redirection:
```bash
echo "This will be logged!" | logoutput
```

File Read:
```bash
logoutput ./test.txt
```