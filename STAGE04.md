## STAGE 3: Creating an EFS file system

---

## What was achieved:
1.  Creating an EFS file system designed to store the wordpress locally for stored media.
2.  This area stores any media for posts uploaded when creating the post as well as theme data.
3.  By storing this on a shared file system, the data can be used across all instances in a consistent way, and it lives on past the lifetime of the instance.

---

### STAGE 4A: Creating an EFS File System
- Modify File System settings

--Creating file System.

- Modify Network Settings
  
  --Configuring the EFS Mount Targets which are the network interfaces in the VPC which the instances will connect with.


### STAGE 4B: Add an fsid to parameter store
Need to add another parameter store value for the file system ID, so that the automatically building instance(s) can load this safely.

### STAGE 4C: Connecting the file system to the EC2 instance & copy data
- Need to install the amazon EFS utilities to allow the instance to connect to EFS.
  ```
  sudo yum -y install amazon-efs-utils
  ```

-  Migrating the existing media content from wp-content into EFS
   -  First, copy the content to a temporary location and make a new empty folder.
   -  Then get the efs file system ID from parameter store
   -  Then add a line to /etc/fstab to configure the EFS file system to mount as /var/www/html/wp-content/
   -  Lastly, copy the origin content data back in and fix permissions

### STAGE 4D: Test that the wordpress app can load the media
- Reboot the ec2 instance 
```
reboot
```
and ensure that wordpress blog is loading, which is now loading the media from EFS.

### STAGE 4E: Update the launch template with the config to automate the EFS part
Updating the launch template is necessary, so that it automatically mounts the EFS file system during its provisioning process. 
This means that, when we add autoscaling, all instances will have access to the same media store, allowing the platform to scale.

---

## What was fixed in this iteration:
- FIXED:: *The application and database built manually, taking time and not allowing automation*
- FIXED:: *Slow configuration and launching of Instance*
- FIXED:: *The database of the application is on an instance; scaling IN/OUT risks this media*
- FIXED:: *The application media and UI store is local to an instance; scaling IN/OUT risks this media*

## Limitations with this configuration:

- Customer Connections are directly to an instance; no health-checks or auto-healing

- IP address hardcoded in db
