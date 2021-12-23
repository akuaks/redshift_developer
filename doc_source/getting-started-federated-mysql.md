# Getting started with using federated queries to MySQL<a name="getting-started-federated-mysql"></a>

To create a federated query to MySQL databases, you follow this general approach: 

1. Set up connectivity from your Amazon Redshift cluster to your Amazon RDS or Aurora MySQL DB instance\. 

   To do this, make sure that your RDS MySQL or Aurora MySQL DB instance can accept connections from your Amazon Redshift cluster\. We recommend that your Amazon Redshift cluster and Amazon RDS or Aurora MySQL instance be in the same virtual private cloud \(VPC\) and subnet group\. This way, you can add the security group for the Amazon Redshift cluster to the inbound rules of the security group for your RDS or Aurora MySQL DB instance\. 

   You can also set up VPC peering or other networking that allows Amazon Redshift to make connections to your RDS or Aurora MySQL instance\.  For more information about VPC networking, see the following\. 
   + [What is VPC peering?](https://docs.aws.amazon.com/vpc/latest/peering/what-is-vpc-peering.html) in the *Amazon VPC Peering Guide*
   + [Working with a DB instance in a VPC ](https://docs.aws.amazon.com/AmazonRDS/latest/UserGuide/USER_VPC.WorkingWithRDSInstanceinaVPC.html) in the *Amazon RDS User Guide*
**Note**  
If your Amazon Redshift cluster is in a different VPC than your RDS or Aurora MySQL instance, then enable enhanced VPC routing\. Otherwise, you might receive timeout errors when you run a federated query\. 

1. Set up secrets in AWS Secrets Manager for your RDS MySQL and Aurora MySQL databases\. Then reference the secrets in AWS Identity and Access Management \(IAM\) access policies and roles\. For more information, see [Creating a secret and an IAM role to use federated queries](federated-create-secret-iam-role.md)\. 
**Note**  
If your cluster uses enhanced VPC routing, you might need to configure an interface VPC endpoint for AWS Secrets Manager\. This is necessary when the VPC and subnet of your Amazon Redshift cluster don't have access to the public AWS Secrets Manager endpoint\. When you use a VPC interface endpoint, communication between the Amazon Redshift cluster in your VPC and AWS Secrets Manager is routed privately from your VPC to the endpoint interface\. For more information, see [Creating an interface endpoint](https://docs.aws.amazon.com/vpc/latest/userguide/vpce-interface.html#create-interface-endpoint) in the *Amazon VPC User Guide*\. 

1. Apply the IAM role that you previously created to the Amazon Redshift cluster\. For more information, see [Creating a secret and an IAM role to use federated queries](federated-create-secret-iam-role.md)\.

1. Connect to your RDS MySQL and Aurora MySQL databases with an external schema\. For more information, see [CREATE EXTERNAL SCHEMA](r_CREATE_EXTERNAL_SCHEMA.md)\. For examples on how to use federated queries, see [Example of using a federated query with MySQL](federated_query_example.md#federated_query_example_mysql)\.

1. Run your SQL queries referencing the external schema that references your RDS MySQL and Aurora MySQL databases\. 