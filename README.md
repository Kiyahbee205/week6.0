# week6.0
Week 5: Scaling, Load Balancing, and DNS

HarborTech Ticket Summary

Ticket TKT-2026-0005 focuses on availability, scaling, traffic distribution, health checks, and DNS failover. Previous promotion traffic reached 92% CPU utilization, and the upcoming promotion is expected to double request volume. The proposed Auto Scaling design is a minimum of 2 instances, a desired capacity of 2, and a maximum of 6.

The ticket also states that an Application Load Balancer has two test targets reported as healthy. A secondary recovery endpoint is available, but Route 53 failover has not been confirmed.

Client Impact

The expected increase in promotion traffic could place additional CPU and request load on the application. If capacity is not available, users could experience slow responses, errors, or an unavailable service.

There is also a risk if client traffic depends on one endpoint. Adding more instances does not automatically distribute traffic between them. A load balancer is needed to distribute requests, and DNS failover requires additional Route 53 configuration and testing.

Provided Ticket Evidence

The following information was provided in the HarborTech ticket and was not personally verified unless specifically stated in the AWS evidence section:

Previous promotion traffic reached 92% CPU utilization.

The upcoming promotion is expected to double request volume.

The proposed Auto Scaling design is minimum 2, desired 2, maximum 6.

The proposed Application Load Balancer reports two test targets as healthy.

A secondary recovery endpoint is available in another approved environment.

Route 53 failover has not been confirmed.

AWS Commands Used

Investigation 1: Auto Scaling Groups

aws autoscaling describe-auto-scaling-groups --query 'AutoScalingGroups[].{Name:AutoScalingGroupName,Min:MinSize,Desired:DesiredCapacity,Max:MaxSize}' --output table

Investigation 2: Target Groups

aws elbv2 describe-target-groups --query 'TargetGroups[].{Name:TargetGroupName,TargetGroupArn:TargetGroupArn,Protocol:Protocol,Port:Port}' --output table

If a target group is returned, the target health command is:

TARGET_GROUP_ARN=$(aws elbv2 describe-target-groups --query 'TargetGroups[0].TargetGroupArn' --output text)
echo "$TARGET_GROUP_ARN"
aws elbv2 describe-target-health --target-group-arn "$TARGET_GROUP_ARN" --query 'TargetHealthDescriptions[].{Target:Target.Id,Port:Target.Port,State:TargetHealth.State,Reason:TargetHealth.Reason}' --output table

Investigation 3: Route 53 Hosted Zones

aws route53 list-hosted-zones --query 'HostedZones[].{Name:Name,Id:Id,Private:Config.PrivateZone}' --output table

AWS Evidence Collected

Investigation 1: Auto Scaling

Actual AWS result: No Auto Scaling Groups were returned by the command in my assigned AWS environment.

This means I did not personally verify an existing Auto Scaling Group or existing minimum, desired, and maximum capacity values. The 2 / 2 / 6 configuration is therefore treated as provided ticket evidence/proposed design, not as an AWS configuration that I verified.

Investigation 2: Target Groups and Target Health

Actual AWS result: The target-group result was not captured in the evidence available to me. Therefore, I cannot claim that I personally verified a target group or target health in AWS.

The ticket's statement that two test targets are healthy remains provided ticket evidence. I did not substitute another student's ARN or create a target group to produce evidence.

Investigation 3: Route 53

Actual AWS result: The Route 53 hosted-zone result was not captured in the evidence available to me. Therefore, I cannot claim that I personally verified a hosted zone, failover record, or Route 53 health check.

The secondary recovery endpoint is provided ticket evidence. Route 53 failover remains an item requiring validation.

Virtualization Connection

Auto Scaling and load balancing are useful with virtual compute because additional virtual instances can be added when demand increases instead of depending on one physical server. Capacity can expand or contract based on operational needs. A load balancer can then distribute client requests across the available virtual instances.

This supports scalability and availability without requiring the workload to depend on one physical machine. However, scaling capacity, distributing traffic, checking health, and providing DNS failover are separate controls.

Operational Analysis

The ticket evidence supports a real capacity concern because previous promotion traffic reached 92% CPU and the upcoming promotion is expected to double request volume. The proposed 2 / 2 / 6 Auto Scaling range is intended to maintain at least two instances, normally run two, and allow scaling up to six when demand increases.

My AWS evidence confirms that the assigned environment returned no Auto Scaling Groups, so I could not verify that the proposed scaling configuration currently exists.

A load balancer would provide traffic distribution, while a target group identifies the targets available to the load balancer. Target health indicates whether the targets are responding to health checks. Healthy targets alone do not prove that the application can survive a DNS or endpoint failure because clients still need a working endpoint to reach the load balancer or application.

The ticket states that a secondary recovery endpoint exists, but my available AWS evidence does not confirm Route 53 failover. Before DNS failover can be considered ready, HarborTech should verify primary and secondary Route 53 records, failover routing, health checks, and an actual failover test.

Recommendation

For capacity, I recommend that HarborTech evaluate the proposed 2 / 2 / 6 Auto Scaling design because the ticket shows a significant expected increase in traffic. Any production scaling changes should follow the approved change process.

For traffic distribution, HarborTech should use an Application Load Balancer and target group so client requests can be distributed across healthy instances. Target health should be monitored continuously.

For health behavior, HarborTech should monitor instance health, target health, CPU utilization, request volume, response time, and application errors. Health checks should test meaningful application availability rather than only confirming that an instance responds.

For DNS routing, HarborTech should verify Route 53 configuration before claiming failover readiness. The primary and secondary endpoints should have appropriate health checks and failover routing, followed by a controlled test to confirm that clients are directed to the recovery endpoint when the primary endpoint fails.

The evidence currently supports the need for better capacity and availability planning, but it does not prove that Auto Scaling, load balancing, or Route 53 failover are fully configured in my assigned environment.

Escalation Notes

Production changes to Auto Scaling, the Application Load Balancer, target groups, Route 53 records, health checks, or failover settings should be completed only through the approved change and escalation process.

The following items require additional verification or authorized support:

Whether the proposed Auto Scaling Group has been created in the correct environment.

Whether the Application Load Balancer and target group are configured.

Whether both application targets are actually healthy in AWS.

Whether Route 53 hosted zones and failover records are configured.

Whether the secondary recovery endpoint is reachable and tested.

Whether a controlled failover test has been approved.

No resources were created or modified during this investigation to manufacture evidence.

Lessons Learned

Week 5 showed me that cloud availability requires more than simply adding servers. Capacity, traffic distribution, health checks, and DNS routing each have different purposes. I also learned the importance of separating ticket evidence from AWS evidence.

If a command does not return a resource, that result should be documented instead of creating or borrowing a resource just to produce evidence. This makes the investigation more accurate and helps operations teams understand what still needs to be verified.

Professional Vocabulary

Elasticity: The ability of a cloud environment to increase or decrease resources as demand changes.

Scalability: The ability of a system to handle increased workload by adding resources.

Load balancer: A service that distributes client traffic across available application targets.

Target group: A group of targets, such as EC2 instances, that a load balancer sends traffic to.

Health check: A test used to determine whether a target or service is responding as expected.

Auto Scaling group: A collection of instances managed together so capacity can be adjusted based on configured limits and conditions.

Launch template: A reusable configuration that defines how new EC2 instances should be launched.

Desired capacity: The number of instances an Auto Scaling group attempts to maintain during normal operation.

Minimum capacity: The lowest number of instances an Auto Scaling group should maintain.

Maximum capacity: The highest number of instances an Auto Scaling group is allowed to maintain.

Route 53: AWS's DNS service used to route users to applications and services.

Failover: A process that directs traffic to a backup resource when the primary resource is unavailable.

Target health: The health status reported for targets based on configured health checks.
