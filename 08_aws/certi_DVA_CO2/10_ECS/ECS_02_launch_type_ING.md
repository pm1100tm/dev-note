# ECS Launcy Type

- v46. 320 ~ 321

Amazon ECS - EC2 Launch Type
• ECS = Elastic Container Service
• Launch Docker containers on AWS =
Launch ECS Tasks on ECS Clusters
• EC2 Launch Type: you must provision
& maintain the infrastructure (the EC2
instances)
• Each EC2 Instance must run the ECS
Agent to register in the ECS Cluster
• AWS takes care of starting / stopping
containers

Amazon ECS – Fargate Launch Type
• Launch Docker containers on AWS
• You do not provision the infrastructure
(no EC2 instances to manage)
• It’s all Serverless!
• You just create task definitions
• AWS just runs ECS Tasks for you based
on the CPU / RAM you need
• To scale, just increase the number of
tasks. Simple - no more EC2 instances
