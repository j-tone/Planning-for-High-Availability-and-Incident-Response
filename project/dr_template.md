# Infrastructure

## AWS Zones
Primary site:
us-east-2a
us-east-2b
us-east-2c

Secondary site:
us-west-1b
us-west-1c

## Servers and Clusters

### Table 1.1 Summary
| Asset                                     | Purpose                                                                             | Size      | Qty     | DR                                                                                            |
|-------------------------------------------|-------------------------------------------------------------------------------------|-----------|---------|-----------------------------------------------------------------------------------------------|
| EC2 instances                             | To provide servers where the application code will be hosted and ran                | t3.micro  | 6       | Yes; 3 in DR, replicated in a separate region (us-west-1)                                     |
| EKS clusters                              | To help deploy the application and house the monitoring stack                       | t3.medium | 4       | Yes; 2 in DR, replicated, located in another region (us-west-1)                               |
| VPC                                       | Stores network configuration for a region                                           |           | 2       | Yes; 1 in DR, each configured for a different region                                          |
| ALBs                                      | To manage the load (incoming/outgoing requests) of the application                  |           | 2       | Yes; 1 in DR, replicated, located in a separate region (us-west-1)                            |
| SQL Clusters                              | To store data for the application                                                   | t3.medium | 4 nodes | Yes; 2 nodes in DR, zone1 replicated to zone2. Each cluster with multiple availability zones. |
| Monitoring stack (Prometheus and Grafana) | Gathering metrics for the application and setting up SLOs and monitoring dashboards |           | 1       | 1 configuration for both sites, but replicated between EKS clusters and ALBs.                 |
| AMI Images (ubuntu x86 x64)               | OS for our EC2 instances                                                            |           | 1       | Image is stored in a completely different site (us-east-1) and owned by Amazon                                    |


### Descriptions
More detailed descriptions of each asset identified above.
EC2 instances: virtual servers that run the application code. These instances are deployed with the AMI image listed above. These are replicated between regions and are used with each availability zone in each region.
EKS clusters: Kubernetes nodes that house the monitoring stack; prometheus and grafana are installed here and metrics are exported via the node exporter. These clusters are replicated between regions.
VPC: Networking layer for our application. Configures security groups, what IPs can access the application, and what IPs the application uses.
ALBs: Load balancers are the entry point to each zone. At a point of site failure, the DNS can be pointed to the secondary load balancer to point the frontward facing IPs that the customer uses to the secondary site, so that the customer does not experience an outage.
SQL clusters: Stores application data with SQL database. Each SQL node in each site has a worker node and reader node. These are replicated between regions, and backups are held for 5 days.
Monitoring Stack: Prometheus and Grafana reside on the EKS clusters and gather/display metrics for the application. Metrics are exported via the node exporter.
AMI Images: Amazon-owned images for the EC2 instances. This is operating system software designed for use on EC2.



## DR Plan
### Pre-Steps:
1. Ensure application assets are replicated in DR site (zone2)
    a. EC2, EKS, ALB should be replicated from zone1 to zone2
2. Verify that SQL nodes are replicated to secondary site (zone2)
    a. Zone1 should be primary node, zone2 should be replicated node

## Steps:
1. Point DNS to cloud load balancer. 
2. Failover DNS to secondary site 
3. Make secondary database clusters the primary clusters (if not automatically taken over as primary)
