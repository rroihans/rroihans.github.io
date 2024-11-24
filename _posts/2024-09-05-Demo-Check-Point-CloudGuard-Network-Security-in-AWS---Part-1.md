---
title: Demo Check Point CloudGuard Network Security in AWS Part 1
date: 2024-11-19 22:05:12 +/-TTTT
categories: [GENERAL]
---

# Tentang
Pada #BercaTechSummit2024 kemarin saya ada kesempatan untuk mewakili Berca Booth Solution untuk Check Point, jadi disamping mempersiapkan materi saya juga siapkan demo. Solusi yang dibawakan pada booth yang kami bawakan adalah CloudGuard Network Security jadi demo yang kebayang oleh saya adalah deploy Firewall di AWS dan beberapa EC2 sebagai komputer "LAN" nya yang diproteksi oleh #CheckPoint.

![IMG_1122.JPG](/assets/2024-09-05-Demo-Check-Point-CloudGuard-Network-Security-in-AWS---Part-1/IMG_1122.JPG)

Berca Booth Solution

![IMG_0998.jpg](/assets/2024-09-05-Demo-Check-Point-CloudGuard-Network-Security-in-AWS---Part-1/IMG_0998.jpg)

Check Point Booth

![IMG_0777.jpg](/assets/2024-09-05-Demo-Check-Point-CloudGuard-Network-Security-in-AWS---Part-1/IMG_0777.jpg)

# Topologi

![cp-bts-aws-demo.drawio (2).png](/assets/2024-09-05-Demo-Check-Point-CloudGuard-Network-Security-in-AWS---Part-1/cp-bts-aws-demo.drawio%20(2).png)

## Dictionary
- CP = Check Point
- AWS = Amazon Web Services
- CGNS = CloudGuard Network Security
- SG = Security Gateway (Firewall)
- SMS = Security Management Server (SMS)
- VPC = Virtual Private Cloud
- BTS = Berca Tech Summit

# Instalasi

**Note: Jika tidak dirincikan maka field/pilihannya adalah default**

## Enable Smart-1 Cloud

Masuk ke portal.checkpoint.com > Quantum > Security Management & Smart-1 Cloud

![Screenshot 2024-11-22 151247.png](/assets/2024-09-05-Demo-Check-Point-CloudGuard-Network-Security-in-AWS---Part-1/Screenshot%202024-11-22%20151247.png)

- Start free trail > Start a demo
- Connect Gateways > Create a New Gateway

Gateway Name: **cp-aws-bts-demo**

![Screenshot 2024-11-22 151412.png](/assets/2024-09-05-Demo-Check-Point-CloudGuard-Network-Security-in-AWS---Part-1/Screenshot%202024-11-22%20151412.png)

Copy dan simpan tokennya untuk digunakan nanti.

** Maka akan tertulis "Waiting for Connection..." gambar diatas adalah sesudah connect dengan SG. Statusnya completed.

## Buat VPC

Sebelum deploy CP SG dan komponen lainnya, kita buat VPC terlebih dahulu:

Navigasi ke VPC > Pilih **VPC and more**, dan isi:

| Key                          | Value               |
| ---------------------------- | ------------------- |
| Name tag                     | cp-aws-bts-demo-vpc |
| IPv4 CIDR Block              | 10.0.0.0            |
| Number of Availability Zones | 1                   |
| Number of public subnets     | 1                   |
| Number of private subnets    | 1                   |
| VPC Endpoints                | None                |

![Screenshot 2024-11-22 152818 1.png](/assets/2024-09-05-Demo-Check-Point-CloudGuard-Network-Security-in-AWS---Part-1/Screenshot%202024-11-22%20152818.png)

## Buat Key Pair

Navigasi ke EC2 > Network & Security > Key Pairs > Create Key Pair

Nama : **cp-aws-bts-lab**  
Type : **rsa**

## AWS Marketplace

Untuk bisa pakai CGNS kita perlu subscribe melalui AWS Marketplace.  
[CloudGuard Network Security with Threat Prevention & SandBlast BYOL: AWS Marketplace](https://aws.amazon.com/marketplace/pp/prodview-eqq52wje3qy5e?applicationId=AWSMPContessa&ref_=beagle&sr=0-1)

![Screenshot 2024-11-22 150618.png](/assets/2024-09-05-Demo-Check-Point-CloudGuard-Network-Security-in-AWS---Part-1/Screenshot%202024-11-22%20150618.png)

Pilih **I'll do this later**.

![pasted_image-1.png](/assets/2024-09-05-Demo-Check-Point-CloudGuard-Network-Security-in-AWS---Part-1/pasted_image-1.png)

## Deploy Check Point Security Gateway

Untuk deploy Firewall check point di AWS adalah menggunakan CloudFormation yang sudah dibuat oleh CP.  

Linknya > [sk111013 - AWS CloudFormation Templates](https://support.checkpoint.com/results/sk/sk111013)

Lalu pilih **CloudGuard Network for AWS Single Gateway** dan klik Launch Stack

![pasted-image-2.png](/assets/2024-09-05-Demo-Check-Point-CloudGuard-Network-Security-in-AWS---Part-1/pasted-image-2.png)

Pilih Stack untuk deploy dengan **existing VPC**.

Kemudian paramater-parameternya saya isi seperti di bawah, yang tidak saya isi merupakan default/tidak saya rubah.

| Key                       | Value                            |
| ------------------------- | -------------------------------- |
| Stack name                | cp-aws-bts-lab                   |
| VPC                       | cp-aws-bts-demo-vpc              |
| Public Subnet             | Public Subnet yang sudah dibuat  |
| Private Subnet            | Private Subnet yang sudah dibuat |
| Gateway name              | cp-gateway-bts-demo              |
| Instance type             | c5.xlarge                        |
| Key name                  | cp-aws-bts-lab                   |
| Gateway Version & license | R81.20 BYOL                      |
| Gateway SIC key           | P@ssw0rd                         |
| Smart-1 Cloud Token       | Paste the token                  |

Lalu **Create Stack**. Dan pastikan seperti ini:

![Screenshot 2024-10-29 104009.png](/assets/2024-09-05-Demo-Check-Point-CloudGuard-Network-Security-in-AWS---Part-1/Screenshot%202024-10-29%20104009.png)

## Route Tables

Balik ke VPC > Route tables > Edit routes:

![pasted-image-3.png](/assets/2024-09-05-Demo-Check-Point-CloudGuard-Network-Security-in-AWS---Part-1/pasted-image-3.png)

- Add route
- Destination: 0.0.0.0/0
- Target: Network Interface  
  - Pilih Internal Network Interface dari SG

![pasted-image-4.png](/assets/2024-09-05-Demo-Check-Point-CloudGuard-Network-Security-in-AWS---Part-1/pasted-image-4.png)

Terus **Save**

## Connect SG to SMS

Navigasi ke CP Portal > Smart-1 Cloud > Connect Gateways  
Sekarang statusnya menjadi ==Pending Trust (SIC) Establishment==.  
Navigasi ke Manage & Settings > klik **Open Streamed SmartConsole**

- klik **Communication**
- SIC password: P@ssw0rd
- klik **Initialize**
- Pastikan **Trust Established.** dan klik **OK**
- Setelah itu **Get Topology Results**
- Dan **OK**

### Create Access and NAT Policy

Navigasi ke Security Policies, ubah policy **Cleanup Rule** menjadi ==ANY-ANY-ACCEPT== dengan Logging.

Setelah itu Publish & Install dan Install policynya.
