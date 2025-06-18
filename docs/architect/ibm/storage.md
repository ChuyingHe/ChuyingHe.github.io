# 1. Object Storage
## 1.1 Cloud Object Storage
Provides flexible, cost-effective, and scalable cloud storage for unstructured data.

## 1.2 MinIO (By MinIO)
MinIO offers high-performance, S3 compatible object storage. Native to Kubernetes, compatible with every public cloud.

## 1.3 FileMage Gateway (By Filemage LLC)
FileMage Gateway is an SFTP & FTP server backed by IBM Cloud Object Storage, featuring permission management and more.


# 2. File Storage
## 2.1 File Storage for Classic
Fast and flexible NFS-based file storage with capacity options from 20GB to 12TB.

## 2.2 File Storage for VPC
Create a shared storage that you can share across multiple Virtual Servers within your VPC.

## 2.3 IBM Storage Scale
Automate HPC cluster deployments on IBM Cloud with BYOL IBM Storage Scale, a clustered file system.


# 3. Block Storage
## 3.1 Block Storage for Classic
Persistent iSCSI based storage with high-powered performance and capacity up to 12TB.

## 3.2 Block Storage for VPC
Persistent storage for use as boot and data storage for Virtual Servers in a VPC network.

## 3.3 Block Storage Snapshots for VPC
Back up block storage volumes to IBM Cloud Object Storage with this regional snapshot service.



# Block vs FileSystem
[reference](https://www.youtube.com/watch?v=PmxWTTpXNLI&ab_channel=IBMTechnology)

- Block Storage（块存储）： 块存储将数据 **按固定大小（通常为 512B 或 4KB）分为多个 块/block** 存储。每个块有唯一 ID，可随机访问。它不理解文件结构，仅处理原始数据块。
- File System Storage（文件系统）： 文件系统是一种 数据组织方式，提供了目录结构、文件权限、元数据等。文件存储以路径方式访问，文件系统管理实际数据块的位置。
    <img src="./../imgs/fs.png" width=300/>



|对比项 | Block Storage | File System|
|:-|:-|:-|
|存储单位 | 数据块（raw block） | 文件和目录|
|管理者 | 操作系统/用户自己挂载后管理 | 文件系统负责管理结构|
|访问方式 | 按 block 地址访问 | 按路径访问|
|示例 | Amazon EBS、iSCSI、LUN | NFS、SMB、Amazon EFS|
|Mounting Level|- Virtual OS Level <br/> - Hypervisor Level |- Virtual OS Level |
|典型用途 | - OS - 数据库（如 MySQL、PostgreSQL）<br/>- 虚拟机磁盘（如 VMware、KVM）<br/>- 高性能应用程序 | - 共享文件目录（如企业网盘）<br/>- 日志服务器<br/>- Web 服务器文件资源<br/>- 办公文件共享（如 NAS）|

