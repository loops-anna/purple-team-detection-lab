   # Detection Rules (Blue Team)
   Wazuh detection rules
#!/bin/bash

# BLUE TEAM PROJECT
# Simulated Authentication Event Generator

LOG_DIR="$HOME/blue-team-lab"
LOG_FILE="$LOG_DIR/simulated_auth.log"

mkdir -p "$LOG_DIR"

echo "Starting authentication event simulation..."

for i in {1..6}
do
    TIMESTAMP=$(date '+%b %e %T')

    echo "$TIMESTAMP labvm sshd[1234]: Failed password for invalid user admin from 192.0.2.25 port 40022 ssh2" >> "$LOG_FILE"

    echo "Event $i recorded successfully."

    sleep 1
done

echo "Simulation completed!"
echo "Log file: $LOG_FILE”
