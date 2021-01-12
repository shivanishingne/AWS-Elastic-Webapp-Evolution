## STAGE 2: Automating the build of the app using a Launch Template

---

## What was achieved:
1.  Creating a launch template which can automate the build of WordPress.
2.  Setting up a bootstrapped or AMI-baked build to function effectively for the automation/self healing.
3.  Launch Template contains build instructions for instances improving build time significantly.

---

### STAGE 2A: Creating the Launch Template
- Create new launch template. Launch Template contents:
  - AMI:    `Amazon Linux 2 AMI (HVM), SSD Volume TYpe, Architecture: 64-bit (x86)`
  - Instance Type:  `t2.micro` (free-tier)
  - Key pair:   `Don't include in launch template`
  - Network settings:   `vpc` ; and select ur security grp.

### STAGE 2B: Adding user data
For adding configuration that will build the instance.
- Added the user data in the `User Data` box. Ensure to add an extra line in the end.

### STAGE 2C: Launching an instance using the template
- Select required network settings (vpc/subnet etc)
- Add tag

### STAGE 2D: Testing
- Select the instance and copypaste the IPv4 public IP in new tab.
- Perform initial configurations to make a post.

---

## What was fixed in this iteration:
- FIXED:: *The application and database built manually, taking time and not allowing automation*
- FIXED:: *Slow configuration and launching of Instance*

## Limitations with this configuration:
- The database of the application is on an instance; scaling IN/OUT risks this media

- The application media and UI store is local to an instance; scaling IN/OUT risks this media

- Customer Connections are directly to an instance; no health-checks or auto-healing

- IP address hardcoded in db
