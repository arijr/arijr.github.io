---
layout: post
title: How to Change Docker's Default Directory on Linux
date: 2025-03-03 01:16 -0300
categories: [infra, devops, docker]
tags: [infra, devops, docker]     # TAG names should always be lowercase

author: arijr                     
# for single entry
# or
#authors: [<author1_id>, <author2_id>]   # for multiple entries
toc: true
math: true
mermaid: true
---
![Desktop View](https://cdn.shortpixel.ai/spai/q_lossy+to_auto+ret_img/linuxiac.b-cdn.net/wp-content/uploads/2024/05/change-docker-data-dir-03.jpg){: width="972" height="589" }

## How to Change Docker's Default Directory on Linux

By default, Docker stores its data, including images, containers, volumes, and networks, in the `/var/lib/docker` directory. However, you may want to change this directory to a different location, such as a larger disk or a specific storage partition. This tutorial will guide you through the process of changing Docker's default directory on a Linux system.

---

### Step 1: Confirm the Current Docker Directory
Before making any changes, it’s important to verify the current location where Docker stores its data. You can do this by running the following command:

```bash
docker info | grep 'Docker Root Dir'
```

**Explanation**:  
- `docker info`: Displays detailed information about the Docker installation.  
- `grep 'Docker Root Dir'`: Filters the output to show only the line containing the Docker root directory.  
This command will output something like `Docker Root Dir: /var/lib/docker`, confirming the current storage location.

---

### Step 2: Create or Edit the Docker Configuration File
Docker's configuration file, `daemon.json`, is used to customize Docker's behavior. You need to add a `data-root` entry to this file to specify the new directory.

#### Option 1: Manually Edit the File
1. Open the file `/etc/docker/daemon.json` in a text editor (e.g., `nano` or `vim`):
   ```bash
   sudo nano /etc/docker/daemon.json
   ```
2. Add the following JSON configuration to specify the new directory:
   ```json
   {
     "data-root": "/path/to/new/docker-data"
   }
   ```
   Replace `/path/to/new/docker-data` with your desired directory path.

#### Option 2: Use a Command to Append the Configuration
Alternatively, you can use the `echo` command to append the configuration to the file:
```bash
echo '{ "data-root": "/path/to/new/docker-data" }' | sudo tee /etc/docker/daemon.json
```

**Explanation**:  
- `echo`: Outputs the JSON configuration.  
- `| sudo tee`: Redirects the output to the `daemon.json` file with root privileges.  
- `/etc/docker/daemon.json`: The configuration file where Docker reads its settings.

---

### Step 3: Stop the Docker Daemon
Before moving Docker's data, you need to stop the Docker service to ensure no files are in use.

```bash
sudo systemctl stop docker
```

**Explanation**:  
- `systemctl stop docker`: Stops the Docker service. This ensures that no containers, images, or other Docker resources are actively using the `/var/lib/docker` directory.

---

### Step 4: Verify That Docker Is Stopped
To confirm that Docker has been stopped, check for running Docker processes:

```bash
ps aux | grep -i docker | grep -v grep
```

**Explanation**:  
- `ps aux`: Lists all running processes.  
- `grep -i docker`: Filters the list to show only Docker-related processes.  
- `grep -v grep`: Excludes the `grep` command itself from the results.  
If Docker is stopped, this command should return no output.

---

### Step 5: Copy Docker Data to the New Directory
Use the `rsync` command to copy all Docker data from the old directory to the new one:

```bash
sudo rsync -axPS /var/lib/docker/ /path/to/new/docker-data
```

**Explanation**:  
- `rsync`: A tool for efficiently copying and synchronizing files.  
- `-a`: Archive mode, which preserves permissions, timestamps, and symbolic links.  
- `-x`: Ensures that `rsync` stays within the same filesystem.  
- `-P`: Shows progress during the transfer.  
- `-S`: Handles sparse files efficiently.  
- `/var/lib/docker/`: The source directory (current Docker data).  
- `/path/to/new/docker-data`: The destination directory (new Docker data location).

---

### Step 6: Restart the Docker Daemon
After copying the data, restart Docker to apply the changes:

```bash
sudo systemctl start docker
```

**Explanation**:  
- `systemctl start docker`: Starts the Docker service using the new configuration.

---

### Step 7: Confirm the New Docker Directory
Verify that Docker is now using the new directory:

```bash
docker info | grep 'Docker Root Dir'
```

**Explanation**:  
This command should now output the new directory path, confirming that the change was successful.

---

### Step 8: Check Running Containers
Ensure that your containers are running correctly after the change:

```bash
docker ps
```

**Explanation**:  
- `docker ps`: Lists all running containers. If everything is working, you should see your containers as expected.

---

### Step 9: Remove the Old Docker Data (Optional)
Once you’ve confirmed that everything is working correctly, you can safely delete the old Docker data to free up space:

```bash
sudo rm -r /var/lib/docker
```

**Explanation**:  
- `rm -r`: Recursively removes the specified directory and its contents.  
- `/var/lib/docker`: The old Docker data directory.  
**Warning**: Only perform this step after confirming that the new directory is fully functional.

---

### Summary
By following these steps, you’ve successfully changed Docker's default directory to a new location. This process involves updating the Docker configuration, stopping the Docker service, copying data to the new location, and restarting Docker. Always verify the changes and ensure your containers are functioning correctly before removing the old data.

--- 
