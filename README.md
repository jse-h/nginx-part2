# Setting up the server

## Part 1 - Server Setup

This server setup uses the Arch Linux distribution. You will also be required to clone this git repository to get the two server setup helper bash scripts.

### User and File Setup

The `server_setup_helper` file is a simple bash script that will help create our system user and starting files. This script needed to be executed using `sudo`.

To run the script, copy and run the command below while in the same directory as your script:
```bash
sudo ./server_setup_helper
```

```bash
#!/bin/bash

# Creating the system user named `webgen`
echo "Creating User 'webgen'..."
sudo useradd -r -d /var/lib/webgen -s /usr/sbin/nologin webgen

# Creating webgen's home directory
echo "Creating 'webgen' home directory..."
sudo mkdir -p /var/lib/webgen

# Creating subdirectories
echo "Creating directory structure (bin, documents, HTML)"
sudo mkdir -p /var/lib/webgen/bin /var/lib/webgen/HTML
sudo mkdir -p /var/lib/webgen/documents

# Creating nested files
echo "Setting up files..."
sudo touch /var/lib/webgen/bin/generate_index
sudo touch /var/lib/webgen/HTML/index.html
sudo touch /var/lib/webgen/documents/file-one
sudo touch /var/lib/webgen/documents/file-two
```

Next, you'll have to edit some of the files nested in `webgen`'s home directory. You will have to be using your preferred text editor, in this repository I will be using the `neovim` package as my text editor.

1. `generate_index` file

Open the `generate_index` script with the command below:
```
sudo nvim /var/lib/webgen/bin/generate_index
```

Copy and save the contents below to the script. This script takes system information and displays it and outputs it to our `index.html` file.
```bash
#!/bin/bash

set -euo pipefail

# this is the generate_index script
# you shouldn't have to edit this script

# Variables
KERNEL_RELEASE=$(uname -r)
OS_NAME=$(grep '^PRETTY_NAME' /etc/os-release | cut -d '=' -f2 | tr -d '"')
DATE=$(date +"%d/%m/%Y")
PACKAGE_COUNT=$(pacman -Q | wc -l)
PUBLIC_IP_ADRESS=$(ip -4 a show dev eth0 | grep inet | awk '{print $2}' | cut -d/ -f1)
OUTPUT_DIR="/var/lib/webgen/HTML"
OUTPUT_FILE="$OUTPUT_DIR/index.html"

# Ensure the target directory exists
if [[ ! -d "$OUTPUT_DIR" ]]; then
    echo "Error: Failed to create or access directory $OUTPUT_DIR." >&2
    exit 1
fi

# Create the index.html file
cat <<EOF > "$OUTPUT_FILE"
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>System Information</title>
</head>
<body>
    <h1>System Information</h1>
    <p><strong>Kernel Release:</strong> $KERNEL_RELEASE</p>
    <p><strong>Operating System:</strong> $OS_NAME</p>
    <p><strong>Date:</strong> $DATE</p>
    <p><strong>Number of Installed Packages:</strong> $PACKAGE_COUNT</p>
    <p><strong>Public IP address of server:</strong> $PUBLIC_IP_ADRESS</p>
</body>
</html>
EOF

# Check if the file was created successfully
if [ $? -eq 0 ]; then
    echo "Success: File created at $OUTPUT_FILE."
else
    echo "Error: Failed to create the file at $OUTPUT_FILE." >&2
    exit 1
fi
```
2. `index.html` file

There are no changes required to the `index.html` file. We will be using the `generate_index` script to build the HTML file.

3. `file-one` and `file-two` files

Simply using your text editor, provide sample text for this plain text file. The purpose of this file is to contain some text so you confirm the files are downloadable so the server is effectively acting as a file server. The commands below will open each file:
```
sudo nvim /var/lib/webgen/documents/file-one
sudo nvim /var/lib/webgen/documents/file-two
```

In `file-one`
```
This is file one!
```

In `file-two`
```
This is file two!
```

#### Changing ownerships and permissions

This last step of the user and file setup is to enable permissions and ownerships for some of the files created.

Changing ownership to `webgen`:
```
sudo chown -R webgen:webgen /var/lib/webgen
```

Changing permissions for `webgen`:
```
sudo chmod 770 -R /var/lib/webgen
```

Giving execute permissions to the `generate_index` script:
```
sudo chmod +x /var/lib/webgen/bin/generate_index
```

### Service and Timer Setup

Run unit file script
```bash
#!/bin/bash
SERVICE_PATH="/etc/systemd/system/generate_index.service"
TIMER_PATH="/etc/systemd/system/generate_index.timer"

sudo touch $SERVICE_PATH
sudo touch $TIMER_PATH

# Creating generate_index.service file using here document
if [[ -f "$SERVICE_PATH" ]]; then
	echo "Creating service file..."
	cat << EOF1 > "$SERVICE_PATH" << EOF2 > "$TIMER_PATH"
	[Unit]
	Description=Generate Index Script Service
	Wants=network-online.target
	After=network-online.target

	[Service]
	User=webgen
	Group=webgen
	ExecStart=/var/lib/webgen/bin/generate_index
	EOF1
fi

# Creating generate_index.timer file using here document
if [[ -f "$TIMER_PATH" ]]; then
	echo "Creating timer file..."
	cat << EOF2 > "$TIMER_PATH"
	[Unit]
	Description=Generate Index Service Timer

	[Timer]
	OnCalendar=*-*-* 05:00:00
	Persistent=true

	[Install]
	WantedBy=timers.target
	EOF2
fi
```

go over contents of script, must use sudo

enabling/starting services and timers

# nginx-setup

install nginx

Using the sites-available and sites-enabled approach for web server

configure nginx.conf
copy paste to configure server block

enable server block with symlink
sudo ln -s /etc/nginx/sites-available/webgen.conf /etc/nginx/sited-enabled/

add /sites-enabled directory to nginx.conf

restart -> start nginx