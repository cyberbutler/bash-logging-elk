FROM ubuntu
ARG ELK_VERSION

# Setup
RUN apt-get -y update
RUN apt-get -y install rsyslog curl sudo openssl

# Install FileBeat
RUN curl -L -O https://artifacts.elastic.co/downloads/beats/filebeat/filebeat-${ELK_VERSION}-amd64.deb
RUN dpkg -i filebeat-${ELK_VERSION}-amd64.deb

# Set the PROMPT_COMMAND env variable
ENV PROMPT_COMMAND '\
RETRN_VAL=$?; \
if [ -f /tmp/lastoutput.tmp ]; \
	then LAST_OUTPUT=$(cat /tmp/lastoutput.tmp); \
	rm /tmp/lastoutput.tmp; \
fi; \
logger -S 10000 -p local6.debug "{\
\"user\": \"$(whoami)\", \
\"path\": \"$(pwd)\", \
\"pid\": \"$$\", \
\"b64_command\": \"$(history 1 | sed "s/^[ ]*[0-9]\+[ ]*//" | base64 -w0 )\", \
\"status\": \"$RETRN_VAL\", \
\"b64_output\": \"$LAST_OUTPUT\"\
}"; \
unset LAST_OUTPUT; \
'

# Add PROMPT_COMMAND to global bash profiles 
RUN echo "export PROMPT_COMMAND='${PROMPT_COMMAND}'" >> /etc/bash.bashrc
RUN echo "export PROMPT_COMMAND='${PROMPT_COMMAND}'" >> /etc/profile

# Create a helper function so that we can log the output of commands when desired with `logoutput <command>`

RUN echo 'logoutput() { output=$(while read input; do echo "$input"; done < "${1:-/dev/stdin}"); echo -e "$output\n"; echo -e "$output" | head -c 10000 | base64 -w0 > /tmp/lastoutput.tmp; return $?; }' >> /etc/bash.bashrc

# Configure rsyslog to log to bash.log
RUN echo 'local6.*    /var/log/bash.log' >> /etc/rsyslog.d/bash.conf

# Configure Logrotate rsyslog to rotate bash.log by inserting rotate statement after /var/log/messages
RUN sed -i '/^\/var\/log\/messages/a /var/log/bash.log' /etc/logrotate.d/rsyslog

# Because we are running in a container, lets disable kernel messages in rsyslog
RUN sed -i 's/module(load="imklog"/# module(load="imklog"/' /etc/rsyslog.conf


# Configure FileBeat
COPY conf/filebeat.yml /etc/filebeat/filebeat.yml

COPY start-services.sh start-services.sh

# Create the operator user with the password "test"
RUN useradd -r -g operator -s /bin/bash --home /tmp -p $(openssl passwd -1 test) operator
# Add operator to sudoers
RUN usermod -aG sudo operator
# Disable sudo coredump
RUN echo "Set disable_coredump false" >> /etc/sudo.conf

CMD ./start-services.sh
