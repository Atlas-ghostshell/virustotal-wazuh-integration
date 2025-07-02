# virustotal-wazuh-integration

#  VirusTotal Integration with Wazuh + Automated Malware Removal

This project showcases how to integrate VirusTotal with Wazuh for real-time malware detection and automated active response.

---

##  Concept

- *Wazuh* monitors directories (e.g., Desktop, Downloads) for new files using FIM with realtime="yes".
- On file creation, Wazuh sends a hash to *VirusTotal* using your API key.
- If VirusTotal flags it as malicious, Wazuh triggers **Rule ID 87105**.
- An *active response script* (remove-threat.sh) is launched to delete the file instantly.

---

##  Files & Structure

###  ossec.conf Snippet

``xml
<integration>
  <name>virustotal</name>
  <api_key>YOUR_API_KEY</api_key>
  <group>syscheck</group>
  <alert_format>json</alert_format>
</integration>

 Active Response Block

<active-response>
  <command>remove-threat</command>
  <location>local</location>
  <rules_id>87105</rules_id>
</active-response>

<command>
  <name>remove-threat</name>
  <executable>remove-threat.sh</executable>
  <timeout_allowed>no</timeout_allowed>
</command>

## Script used to detect and remove threat.

 remove-threat.sh

#!/bin/bash

LOCAL=$(dirname $0)
cd $LOCAL
cd ../

PWD=$(pwd)

read INPUT_JSON
FILENAME=$(echo $INPUT_JSON | jq -r .parameters.alert.data.virustotal.source.file)
COMMAND=$(echo $INPUT_JSON | jq -r .command)
LOG_FILE="${PWD}/../logs/active-responses.log"

if [ "$COMMAND" = "add" ]; then
  printf '{"version":1,"origin":{"name":"remove-threat","module":"active-response"},"command":"check_keys", "parameters":{"keys":[]}}\n'
  read RESPONSE
  COMMAND2=$(echo $RESPONSE | jq -r .command)

  if [ "$COMMAND2" != "continue" ]; then
    echo "$(date '+%Y/%m/%d %H:%M:%S') remove-threat: Aborted for $FILENAME" >> $LOG_FILE
    exit 0
  fi
fi

rm -f "$FILENAME"
if [ $? -eq 0 ]; then
  echo "$(date '+%Y/%m/%d %H:%M:%S') remove-threat: Removed $FILENAME" >> $LOG_FILE
else
  echo "$(date '+%Y/%m/%d %H:%M:%S') remove-threat: Failed to remove $FILENAME" >> $LOG_FILE
fi

exit 0


---

## Testing

Tested using the EICAR test malware.

Confirmed that the file was:

1. Detected by VirusTotal


2. Logged by Wazuh (Rule 87105)


3. Deleted instantly by the script





---


![Screenshot 2025-07-02 182454](https://github.com/user-attachments/assets/b0f9d803-3436-4ebe-ae4d-c33a1dafda5e)



---

 Author

Built by Jeffrey & Atlas Maru Shepherd, 2025.

---
