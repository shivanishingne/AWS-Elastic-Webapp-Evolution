## STAGE 5: Adding an ELB and ASG

---

## What was achieved:
1.  Adding an auto scaling group to provision and terminate instances automatically based on load on the system.

### STAGE 5A: Creating the load balancer
- Click Create under `Application Load Balancer`
- Under Listeners `HTTP` and `80` should be selected for Load Balancer Protocol and Load Balancer Port
- Select required VPC and subnets
- In `Configure Routing`, for Target Group choose New Target Group
- Copy dns name to clipboard
  
### STAGE 5B: Creating a new Parameter store value with the ELB DNS name
- In Parameter store in Systems Manager, create new param for the alb and set its value as the dns name above.
  
### STAGE 5C: Update the Launch template so wordpress gets updated with the ELB DNS as its home
- Add the new parameter in the beginning of the user data.
- After creating the template, set it as the default version.

### STAGE 5D: Creating an auto scaling group
- Create new ASG from the ec2 console

### STAGE 5E: Integrating ASG with the ALB
- Load balancers actually work (for EC2) with static instance registrations.
- ASG links with a target group; so any instance provisioned by the ASG is added to the target group, anything terminated is removed.
- Make sure Enable instance scale-in protection is NOT checked

### STAGE 5F: Add scaling
 Added two policies, Scale-in and Scale-out:

 #### SCALEOUT when CPU usage on average is above 40%
- Click Add Policy
- For policy type select `Simple scaling`
- for Scaling Policy name enter `HIGHCPU`
- Click `Create a CloudWatch Alarm`
- Click `Select Metric` & Click `EC2`
- select `Greater`

#### SCALEIN when CPU usage on average is below 40%
- Click Add Policy
- For policy type select `Simple scaling`
- for Scaling Policy name enter `LOWCPU`
- Click `Create a CloudWatch Alarm`
- Click `Select Metric` & Click `EC2`
- select `Lower`

#### Adjust the ASG Values
- Click Details Tab
- Click Edit
- Set Desired 1, Minimum 1 and Maximum 3
- Click Update

### STAGE 5G: Test Scaling & Self Healing
- Simulate some load on the wordress instance
  ```
    stress -c 2 -v -t 3000
  ```
- At some point another instance will be added. This will be auto built based on the launch template- connect to the RDS instance and EFS file system and add another instance of capacity to the platform.
- When one instance on EC2 is terminated, a new instance is provisioned to take the old ones place. This is an example of `self-healing`.


---

## What was fixed in this iteration:
- FIXED:: *The application and database built manually, taking time and not allowing automation*
- FIXED:: *Slow configuration and launching of Instance*
- FIXED:: *The database of the application is on an instance; scaling IN/OUT risks this media*
- FIXED:: *The application media and UI store is local to an instance; scaling IN/OUT risks this media*
- FIXED:: *Customer Connections are to an instance directly ... no health checks/auto healing*
- FIXED:: *The IP of the instance is hardcoded into the database*

