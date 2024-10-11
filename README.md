# PROXMOX-BACKUP-AUTO-LXC-NOTIF-TELEGRAM
PROXMOX BACKUP AUTO LXC NOTIF TELEGRAM
![Logo](https://upload.wikimedia.org/wikipedia/commons/9/92/Logo_Proxmox.svg)

## About
This is a backup system from LXC Proxmox to another LXC Proxmox with an automatic backup system every week and sending notifications to Telegram of the backup results, you can also see the backup results on the website display.


## Setting Steps
1. Proxmox Host Preparation:
Make sure both Proxmox servers can communicate with each other via SSH.
Add a passwordless SSH key between the two Proxmox servers, so that backup automation can run without manual interaction.
On the source Proxmox (Proxmox A):
```bash
  ssh-keygen -t rsa
  ssh-copy-id root@<IP_Proxmox_B>

```

2. Create a Backup Script:
Create a script in Proxmox A to perform an LXC backup and send the backup results to Proxmox B.
Create a script named auto_backup_lxc.sh on Proxmox A.

```bash
#!/bin/bash

# Variabel
LXC_ID=124  # ID LXC yang akan di-backup
BACKUP_DIR=/var/lib/vz/dump  # Lokasi backup di Proxmox asal
TARGET_IP=192.168.11.19  # IP LXC di Proxmox lain
TARGET_DIR=/var/backup/lxc  # Direktori penyimpanan di LXC tujuan
TELEGRAM_TOKEN="YOUR_TOKEN_TELEGRAM"
TELEGRAM_CHAT_ID="YOUR_ID_CHAT_TELEGRAM"

# Melakukan backup
vzdump $LXC_ID --dumpdir $BACKUP_DIR --mode snapshot --compress gzip

# Memindahkan hasil backup ke LXC Proxmox lain
scp $BACKUP_DIR/vzdump-lxc-$LXC_ID*.gz root@$TARGET_IP:$TARGET_DIR

# Jika berhasil, hapus backup yang ada di Proxmox asal
if [ $? -eq 0 ]; then
    echo "Transfer berhasil, menghapus file backup lokal..."
    rm -f $BACKUP_DIR/vzdump-lxc-$LXC_ID*.gz
else
    echo "Transfer gagal, tidak menghapus file backup lokal."
fi

# Mengirim notifikasi ke Telegram
curl -s -X POST https://api.telegram.org/bot$TELEGRAM_TOKEN/sendMessage \
    -d chat_id=$TELEGRAM_CHAT_ID \
    -d text="Backup LXC $LXC_ID selesai dan dipindahkan ke $TARGET_IP:$TARGET_DIR pada $(date +"%Y-%m-%d %H:%M:%S")"
```

Change the LXC_ID, Proxmox B IP, and REMOTE BACKUP DIR as needed.

3. Adding a Cron Job for Automation:
Add a cron job to run this script automatically, for example every week on Sunday at 02:00.

Open crontab:
```bash
crontab -e
```
Add the following lines:
```bash
0 2 * * 0 /path/to/auto_backup_lxc.sh >> /var/log/lxc_backup.log 2>&1
```
This will run the auto_backup_lxc.sh script every Sunday at 02:00 and log it to /var/log/lxc_backup.log .

4. Testing the Script:
Run the script manually to make sure everything is working correctly:
```bash
bash /path/to/auto_backup_lxc.sh
```


## Screenshots

![App Screenshot](https://raw.githubusercontent.com/sulthanullah/PROXMOX-BACKUP-AUTO-LXC-NOTIF-TELEGRAM/refs/heads/main/Screenshot/1.jpg)
![App Screenshot](https://raw.githubusercontent.com/sulthanullah/PROXMOX-BACKUP-AUTO-LXC-NOTIF-TELEGRAM/refs/heads/main/Screenshot/2.jpg)
![App Screenshot](https://raw.githubusercontent.com/sulthanullah/PROXMOX-BACKUP-AUTO-LXC-NOTIF-TELEGRAM/refs/heads/main/Screenshot/3.jpg)
![App Screenshot](https://raw.githubusercontent.com/sulthanullah/PROXMOX-BACKUP-AUTO-LXC-NOTIF-TELEGRAM/refs/heads/main/Screenshot/4.jpg)
## Sourcode Web Monitoring

 - [For Web Monitoring](https://github.com/sulthanullah/Website-Backup-Server-Monitoring-LXC)
