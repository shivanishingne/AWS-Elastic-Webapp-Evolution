## STAGE 3: Separating the database functionality from the EC2 instance

---

## What was achieved:
1.  Splitting out the database functionality from the existing EC2 instance.
2.  Running MariaDB to an RDS instance running the MySQL Engine.
3.  Allow the DB and the Instance to scale independently, and allow the data to be secure past the lifetime of the EC2 instance.

---

### STAGE 3A: Creating an RDS Subnet Group
A subnet group is what allows RDS to select from a range of subnets to put its databases inside

### STAGE 3B: Creating an RDS Instance
Provision an RDS instance using the subnet group to control placement within the VPC.
Normally, in prod, it would be preferred to use multi-az for keeping costs low.

### STAGE 3C: Migrating WordPress data from MariaDB to RDS

- Populate Envt variables in EC2 connect/bash:

--For doing an export of the SQL db running on the local EC2 instance, we run some bash commands to populate variables with the data from Parameter store.

- Take a backup of the local DB

 --Note: in production you shouldn't put the password in the CLI, its a security risk since a ps -aux can see it.

- Restore that Backup into RDS

--Update the endpoint in param store, with the endpoint value of the rds instance.

- Update the DbEndpoint environment variable
- Restore the database export into RDS
- Change the wordpress config file to use RDS


### STAGE 3D: Stopping the MariaDB Service
```
sudo systemctl disable mariadb
sudo systemctl stop mariadb
```

### STAGE 3E: Testing Wordpress
The blog should be working, even though MariaDB on the EC2 instance is stopped and disabled. It is now running using RDS

### STAGE 3F: Updating the LT so it doesn't install
Modify Launch template to restrict start of mariadb.

## What was fixed in this iteration:
- FIXED:: *The application and database built manually, taking time and not allowing automation*
- FIXED:: *Slow configuration and launching of Instance*
- FIXED:: *The database of the application is on an instance; scaling IN/OUT risks this media*

## Limitations with this configuration:

- The application media and UI store is local to an instance; scaling IN/OUT risks this media

- Customer Connections are directly to an instance; no health-checks or auto-healing

- IP address hardcoded in db
