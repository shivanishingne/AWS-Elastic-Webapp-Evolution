## STAGE 7: Cleaning up the account and removing the resources

--- 

#### STAGE 7A - Delete Load Balancer & Target Groups

#### STAGE 7B - Delete Auto Scaling Group

#### STAGE 7C - Delete the EFS File system

#### STAGE 7D - Delete RDS

#### STAGE 7E - Check Progress
- Check that EFS and all EC2 instances, and ASG have finished deleting - hold here until they have both been removed.

#### STAGE 7F - Delete Launch Template

#### STAGE 7G - Wait for RDS to be removed, and delete RDS Snapshot

#### STAGE 7H - Delete Cloud Formation stack

---
STAGE 7 - FINISH