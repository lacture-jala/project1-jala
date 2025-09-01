# VPC1 Setup
1. Create Vpc
2. Create Route Table
3. Create subnet
4. Attach Route Table -----> Subnet
5. Create Ec2 Instance (t2.mirco)

# VPC2 Setup
1. Create Vpc
2. Create Route Table
3. Create subnet
4. Attach Route Table -----> Subnet
5. Create Ec2 Instance (t2.mirco)

```mermaid
graph
    subgraph AWS[AWS Cloud]
        
        subgraph VPC1[test-vpc-1]
            Subnet1[Public Subnet]
            EC2_1[EC2 Instance]
            RT1[Route Table<br>172.16.8.0<br>172.16.1.0<br>172.16.2.0]
            Subnet1 --> EC2_1
            Subnet1 --> RT1
        end

        subgraph VPC2[test-vpc-2]
            Subnet2[Public Subnet]
            EC2_2[EC2 Instance]
            RT2[Route Table<br>172.16.8.0<br>172.16.1.0<br>172.16.2.0]
            Subnet2 --> EC2_2
            Subnet2 --> RT2
        end

        VPC1 ==VPC Peering==> VPC2

    end

```