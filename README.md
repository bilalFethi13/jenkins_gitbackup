#I create a script that push a jenkins backup config every time the machine is stopped.

## Write the Script ##

#!/bin/bash
# Define the backup directory and repository URL
	BACKUP_DIR="/home/adminuser/backup/jenkins/jenkins_gitbackup"
	REPO_URL="https://github.com/bilalFethi13/jenkins_gitbackup.git"

# Step 1: Setup the repository
	sudo git config --global --add safe.directory $BACKUP_DIR
	sudo chown -R $(whoami):$(whoami) $BACKUP_DIR
	cd $BACKUP_DIR

# Step 2:  Clone the repository anew to handle submodules correctly
	git clone --recurse-submodules $REPO_URL $BACKUP_DIR

# Step 3: Copy Jenkins files to the backup directory
	sudo rsync -av --exclude='.git' /var/lib/jenkins/ $BACKUP_DIR/

# Step 4: Handle Git operations
	git config user.name "bilalFethi13"
	git config user.email "bilal13.fethi@gmail.com"

# Step5: Add all changes, including handling submodules
	git add .
	git submodule foreach --recursive 'git checkout . && git add . && git commit -m "Update submodule changes"'
	git commit -m "latest change $(date +"%Y-%m-%d %H:%M:%S")"

# Step6: Push changes, including submodules
	git push --recurse-submodules=on-demand

## Step 7: Make the Script Executable
    chmod +x /home/adminuser/jenkins_gitbackup.sh

## Step 8: Create a systemd Service
    sudo nano /etc/systemd/system/jenkins_gitbackup.service
    ## Add the following content to the file:

        [Unit]
        Description=Jenkins Backup at Shutdown
        DefaultDependencies=no
        Before=shutdown.target

        [Service]
        Type=oneshot
        ExecStart=/home/adminuser/jenkins_gitbackup.sh
        User=adminuser
        RemainAfterExit=true

        [Install]
        WantedBy=shutdown.target

## Step 9: Enable the systemd Service
    sudo systemctl enable jenkins_gitbackup.service
    sudo systemctl start jenkins_gitbackup.service

## Step 10: Ensure the Script is Executable
    sudo chmod +x /home/adminuser/jenkins_gitbackup.sh

## Step 11: Configure Git to Use Credential Store
    git config --global credential.helper store


