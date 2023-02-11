## Amazon Web Services (AWS) Elastic Kubernetes Service (EKS) storage classes
Amazon Web Services (AWS) Elastic Kubernetes Service (EKS) supports several storage classes, which define the type of storage that is available to the containers running in the cluster. The following are the storage classes available in AWS EKS:

- gp2 (General Purpose SSD): This is the default storage class in AWS EKS. It provides a balanced combination of performance and cost-effectiveness for a variety of workloads.
- io1 (Provisioned IOPS SSD): This storage class provides high I/O performance and is suitable for use cases that require low latency and high IOPS.
- sc1 (Cold HDD): This storage class provides low cost storage for infrequently accessed data.
- st1 (Throughput Optimized HDD): This storage class provides low cost storage for large, throughput-intensive workloads.
- fsx (Amazon FSx for Lustre): This storage class provides a high-performance file system that is suitable for compute-intensive workloads, such as HPC and machine learning.

You can choose a storage class based on your specific storage requirements and workload characteristics. Additionally, you can also create and manage custom storage classes for your EKS cluster if you have specific storage requirements that are not met by the built-in storage classes.
