# Comparison
[reference](https://www.youtube.com/watch?v=PmxWTTpXNLI&ab_channel=IBMTechnology)




|对比项 | 🧱 Block Storage | 📁 File System/Storage|📦 Object Storage|
|:-|:-|:-|:-|
|存储内容 | 数据库、操作系统盘 | 文档、代码、共享目录	|非结构化数据（如图片、备份、日志）|
|存储单位 | 数据块（raw block） | 文件和目录|对象（Object）：包含数据 + 元数据 + 唯一标识符（Key）|
|访问方式 | 按 block 地址访问 | 按路径访问|通过 RESTful API。比如：<br/> `curl -X PUT https://s3.example.com/bucket/file.txt -T file.txt`|
|示例 | Amazon EBS、iSCSI、LUN | NFS、CephFS（ODF 提供）、SMB、Amazon EFS|AWS S3、MinIO|
|Mounting Level|- Virtual OS Level <br/> - Hypervisor Level |- Virtual OS Level ||
|典型用途 | - OS - 数据库（如 MySQL、PostgreSQL）<br/>- 虚拟机磁盘（如 VMware、KVM）<br/>- 高性能应用程序 | - 共享文件目录（如企业网盘）<br/>- 日志服务器<br/>- Web 服务器文件资源<br/>- 办公文件共享（如 NAS）|大规模、非结构化数据（如图片、视频、备份）|



# 🧱 Block Storage
- 将存储分成“块”，像一块硬盘一样提供原始设备（裸设备）- 按固定大小（通常为 512B 或 4KB）分为多个 块/block
- 每个块有唯一 ID，可随机访问。
- 它不理解文件结构，仅处理原始数据块。
- 操作系统可以格式化它，安装文件系统（ext4、xfs）等
- 每个块卷只能挂载给一个客户端（Pod）

## IBM产品

### Block Storage for Classic
Persistent iSCSI based storage with high-powered performance and capacity up to 12TB.

### Block Storage for VPC
Persistent storage for use as boot and data storage for Virtual Servers in a VPC network.

### Block Storage Snapshots for VPC
Back up block storage volumes to IBM Cloud Object Storage with this regional snapshot service.



# 📁 File Storage

- 类似传统文件服务器（如 NFS、SMB）
- 提供了目录结构、文件权限、元数据等
- 以“文件”和“目录”的形式组织数据
    <img src="./../imgs/fs.png" width=300/>
- 多个客户端可以挂载到同一个路径，支持共享访问

## IBM产品

### File Storage for Classic
Fast and flexible NFS-based file storage with capacity options from 20GB to 12TB.

### File Storage for VPC
Create a shared storage that you can share across multiple Virtual Servers within your VPC.

### IBM Storage Scale
Automate HPC cluster deployments on IBM Cloud with BYOL IBM Storage Scale, a clustered file system.


# 📦 Object Storage
- 数据以“对象”的形式存储，每个对象包含：

    - 数据本身
    - 元数据
    - 全局唯一 ID

- 不是通过传统文件系统访问，而是通过 RESTful API（如 S3） 访问
- 没有文件目录树，访问是基于 URL 或键值

## IBM产品
### Cloud Object Storage （COS）
Provides flexible, cost-effective, and scalable cloud storage for unstructured data.

### MinIO (By MinIO)
MinIO offers high-performance, S3 compatible object storage. Native to Kubernetes, compatible with every public cloud.

### FileMage Gateway (By Filemage LLC)
FileMage Gateway is an SFTP & FTP server backed by IBM Cloud Object Storage, featuring permission management and more.




