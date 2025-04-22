Steps to Replace NFS with Local Disk for Nextcloud
This document outlines the steps required to replace the NFS-mounted /mnt/data directory in Nextcloud with a locally mounted disk (5TB partition) while maintaining proper data integrity.
Steps:
1. Put Nextcloud in Maintenance Mode
Run the following command to put Nextcloud in maintenance mode:
```bash
sudo -u apache php /var/www/html/nextcloud/occ maintenance:mode --on
```
2. Backup (Optional but Safe)
It is recommended to back up the existing data before switching.
Run the following command to back up the NFS data:
```bash
sudo rsync -avz /mnt/data/ /mnt/nfs-backup/
```
3. Unmount NFS
Unmount the current NFS mount using the following command:
```bash
sudo umount /mnt/data
```
Then, edit `/etc/fstab` to remove or comment out the NFS entry:
```bash
sudo nano /etc/fstab
# Comment out the NFS line like:
# 192.168.x.x:/nfs-share /mnt/data nfs defaults 0 0
```
4. Mount the Local Disk
Assuming the 5TB disk is already partitioned and formatted (ext4), mount it with the following command:
```bash
sudo mount /dev/sdb1 /mnt/data
```
To make this mount persistent after reboot, add it to `/etc/fstab`:
```bash
echo '/dev/sdb1  /mnt/data  ext4  defaults  0 2' | sudo tee -a /etc/fstab
```
5. Restore Data (if backed up)
If you performed a backup in step 2, restore the data with the following command:
```bash
sudo rsync -avz /mnt/nfs-backup/ /mnt/data/
```
6. Fix Permissions
Ensure proper permissions for Nextcloud to function correctly:
```bash
sudo chown -R apache:apache /mnt/data
sudo find /mnt/data -type d -exec chmod 750 {} \;
sudo find /mnt/data -type f -exec chmod 640 {} \;
```
7. Disable Maintenance Mode
Disable maintenance mode after the change:
```bash
sudo -u apache php /var/www/html/nextcloud/occ maintenance:mode --off
```
8. Test It!
Finally, verify that everything is working:
- Log in to Nextcloud and check if data is accessible.
- Upload/download some files.
- Review logs if needed (`/var/www/html/nextcloud/data/nextcloud.log`).
