# Tardigrade

Most organizations use virtual machines (VMs) to isolate processes from a hardware and network perspective.
Beyond isolating resources like CPU, disk, and RAM, organizations often need to achieve network-level isolation.
This involves defining the subnet a specific VM belongs to, assigning IP addresses, configuring firewall rules,
determining internet accessibility, etc... As a result, applications with differing network requirements are typically
provisioned on separate virtual machines. In scenarios where they are provisioned in the same VM, as the configurations are
made at the VM level and not at the process level, configurations made for a system affects all other systems that run within the same
virtual machine, which can represent a security and compliance problem.

In scenarios where multiple systems are running in the same VM there's no well-defined mechanisms to control west and east traffic (inter-process 
communication over the network), therefore as long as systems are running under the same VM they have network access to each other. Consequently, if an 
infected process is running inside the VM it can easily access neighboring systems.

That's how Tardigrade comes to rescue! Tardigrade is an opensource Virtual Private Network (VPC) expired by AWS VPC 
that allow users to isolate logically  processes running within a linux VM. Like traditional corporate networks, Tardigrade VPCs can have
public and private subnets. Private subnets do not have routes to the internet 
gateway, but public subnets do. Tardigrade provides a complete control over virtual
networking environment, including the selection of IP address range, 
creation of subnets, and configuration of route tables and network gateways. 
It also allows to use both IPv4 and IPv6 in your VPC for secure and easy access to 
resources and applications.

The Tardigrade Network Interface (TNI), which is attached to processes, enables seamless migration of IP addresses 
from one process to another. This eliminates the need to update ACLs, DNS entries, security group rules, or 
IP addresses on systems that communicate with the affected process. The concept of TNI addresses is particularly
valuable for designing high-availability workloads. Additionally, TNI can be leveraged in blue/green deployments and 
disaster recovery strategies such as warm standby and backup-and-restore scenarios.

As in traditional data centers, you can have granular control over the north/south traffic by using network access control list (ACLs).
ACLs are stateless, and act as a firewall for associated subnets. To control traffic flow at
process level, use security groups. Security group act as a firewall for processes and they are stateful. 

External connectivity options for Tardigrade VPC include the following:
* An `internet gateway` is a VPC component that allows communication between processes in a VPC and the internet.
* A `NAT gateway` enables instances in a private subnet to connect to the internet

Although Tardigrade is not a container orchestration tool, it integrates seamlessly with them, unlocking a wide range of capabilities not natively 
supported by container runtimes. Tardigrade enables containers running on different machines to communicate using their private IP addresses, 
provided they are within the same VPC, or in VPCs that can comunicate with each other. Additionally, it allows you to define firewall rules for 
processes running within containers. 

Tardigrade also supports multi-tier processes, similar to what is possible with VMs on corporate network. To implement this, you can create subnets for 
each tier, for instance, an application tier and a database tier, and configure the database tier to be accessible only by processes running within 
the application tier. This enhances security and simplifies network management for complex applications. 


## Architecture

Tardigrade consists of two logical components: the data plane and the control plane. Both components run as a single process by default, although they can 
be executed independently. Typically, they operate within the same VM they manage.

The control plane features a monolithic design that encompasses storage handling, message serialization, and network transport.
This component facilitates the registration, access, and security of services deployed within the VPC.

The data plane implements the functionality provided by the control plane and interacts directly with underlying system resources.

Tardigrade uses the Raft consensus algorithm. In a clustered deployment, each Tardigrade instance maintains a replicated state machine that 
stays synchronized via a replicated log. When a request, such as creating a resource, is sent to a follower instance, the request 
is forwarded to the leader instance. The leader is responsible for propagating the change to all non-leader instances.

By design, Tardigrade achieves eventual consistency, meaning replication occurs asynchronously. This ensures that changes will 
propagate across the cluster over time while optimizing for availability and responsiveness.

Similar to other systems that utilize the Raft consensus algorithm, during a network partition, the side without a 
quorum (the minimum number of instances required to elect a leader) operates in read-only mode.

