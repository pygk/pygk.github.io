---
layout: custom
title: EC2 Instance Storage
date: 2022-12-29 02:56:00 +0900
last_modified_at: 2022-12-29 02:56:00 +0900
category: SAA-C02
tags: ["AWS", "SAA-C02", "EC2", "EBS", "EFS"]
published: False
---

> EC2 Instance Storage

## 1. EBS (Elastic Block Store)
### 1.1. What’s an EBS Volume?
* An __EBS (Elastic Block Store) Volume__ is a __network__ drive you can attach to your instances while they run
* It allows your instances to persist data, even after their termination
* __They can only be mounted to one instance at a time__ (at the CCP level)
* They are bound to __a specific availability zone__

* Analogy: Think of them as a “network USB stick” 
* Free tier: 30 GB of free EBS storage of type General Purpose (SSD) or Magnetic per month

* __EBS(Elastic Block Store) 볼륨__ 은 인스턴스가 실행되는 동안 인스턴스에 연결할 수 있는 __네트워크__ 드라이브입니다.
* 인스턴스가 종료된 후에도 데이터를 유지할 수 있습니다.
* __한 번에 하나의 인스턴스에만 마운트할 수 있습니다__(CCP 레벨에서)
* __특정 가용성 영역__ 에 바인딩되어 있습니다.

* 비유: 이를 "네트워크 USB 스틱"이라고 생각하십시오.
* 무료 등급: 범용(SSD) 또는 마그네틱 유형의 무료 EBS 스토리지 월 30GB

```
CCP - Certified Cloud Practitioner - one EBS can be only mounted to one EC2 instance
Associate Level (Solution Architect, Developer, SysOps): "multi-attach" feature for some EBS

CCP - Certified Cloud Practicator - 하나의 EBS는 하나의 EC2 인스턴스에만 마운트할 수 있음
Associate Level(Solution Architect, Developer, SysOps): 일부 EBS의 "다중 연결" 기능
```

### 1.2. EBS Volume
* It’s a network drive (i.e. not a physical drive)
    * It uses the network to communicate the instance, which means there might be a bit of latency
    * It can be detached from an EC2 instance and attached to another one quickly

* It’s locked to an Availability Zone (AZ)
    * An EBS Volume in us-east-1a cannot be attached to us-east-1b
    * To move a volume across, you first need to snapshot it

* Have a provisioned capacity (size in GBs, and IOPS)
    * You get billed for all the provisioned capacity
    * You can increase the capacity of the drive over time

* 네트워크 드라이브(즉, 물리적 드라이브가 아님)입니다.
    * 네트워크를 사용하여 인스턴스를 통신합니다. 즉, 약간의 지연 시간이 있을 수 있습니다.
    * EC2 인스턴스에서 분리하여 다른 인스턴스에 빠르게 연결할 수 있습니다.

* 가용성 영역(AZ)에 잠겨 있습니다.
    * us-east-1a의 EBS 볼륨을 us-east-1b에 연결할 수 없습니다.
    * 볼륨을 이동하려면 먼저 스냅샷을 생성해야 합니다.

* 용량 프로비저닝(GB 및 IOPS 단위 크기)
    * 프로비저닝된 모든 용량에 대해 청구됩니다.
    * 시간이 지남에 따라 드라이브의 용량을 늘릴 수 있습니다.

### 1.3. EBS Volume - Example
![Untitled](/assets/img/aws_saa_c02/20221229_ec2instancestorage_ebs_volume.JPG)

### 1.4. EBS – Delete on Termination attribute (중요)
![Untitled](/assets/img/aws_saa_c02/20221229_ec2instancestorage_ebs_delete.JPG)

* Controls the EBS behaviour when an EC2 instance terminates
    * By default, the root EBS volume is deleted (attribute enabled)
    * By default, any other attached EBS volume is not deleted (attribute disabled)
* This can be controlled by the AWS console / AWS CLI
* __Use case: preserve root volume when instance is terminated__

* EC2 인스턴스가 종료될 때 EBS 동작을 제어합니다.
    * 기본적으로 루트 EBS 볼륨은 삭제됩니다(속성 사용).
    * 기본적으로 연결된 다른 EBS 볼륨은 삭제되지 않습니다(속성 비활성화).
* 이는 AWS 콘솔/AWS CLI에서 제어할 수 있습니다.
* __사용 사례: 인스턴스가 종료될 때 루트 볼륨 유지__


## 2. EBS Snapshots
### 2.1. EBS Snapshots
* Make a backup (snapshot) of your EBS volume at a point in time
* Not necessary to detach volume to do snapshot, but recommended
* Can copy snapshots across AZ or Region

* 특정 시점에 EBS 볼륨의 백업(스냅샷) 생성
* 작업관리 스냅샷에 볼륨을 분리할 필요는 없지만 권장
* AZ 또는 지역 간에 스냅샷 복사 가능

![Untitled](/assets/img/aws_saa_c02/20221229_ec2instancestorage_ebs_snapshots.JPG)

### 2.2. EBS Snapshots Features
* __EBS Snapshot Archive__
    * Move a Snapshot to an ”archive tier” that is 75% cheaper
    * Takes within 24 to 72 hours for restoring the archive

* __Recycle Bin for EBS Snapshots__
    * Setup rules to retain deleted snapshots so you can recover them after an accidental deletion
    * Specify retention (from 1 day to 1 year)

* __Fast Snapshot Restore (FSR)__
    * Force full initialization of snapshot to have no latency on the first use ($$$)

* __EBS 스냅샷 아카이브__
    * 스냅샷을 75% 저렴한 "아카이브 계층"으로 이동
    * 아카이브 복원에 24~72시간 소요

* __EBS 스냅샷용 휴지통__
    * 실수로 삭제된 스냅샷을 복구할 수 있도록 삭제된 스냅샷을 보존하는 설정 규칙
    * 보존 지정(1일 ~ 1년)

* __빠른 스냅샷 복원(FSR)__
    * 스냅샷의 전체 초기화로 첫 사용 시 지연 시간 없음($$)

![Untitled](/assets/img/aws_saa_c02/20221229_ec2instancestorage_ebs_snapshots_features.JPG)


## 3. AMI
### 3.1. AMI Overview
* AMI = Amazon Machine Image
* AMI are a __customization__ of an EC2 instance
    * You add your own software, configuration, operating system, monitoring…
    * Faster boot / configuration time because all your software is pre-packaged
* AMI are built for a __specific region__ (and can be copied across regions)
* You can launch EC2 instances from:
    * __A Public AMI__: AWS provided
    * __Your own AMI__: you make and maintain them yourself
    * __An AWS Marketplace AMI__: an AMI someone else made (and potentially sells)

* AMI = Amazon 시스템 이미지
* AMI는 EC2 인스턴스의 __사용자 지정__ 입니다.
    * 자체 소프트웨어, 구성, 운영 체제, 모니터링을 추가할 수 있습니다.
    * 모든 소프트웨어가 사전 패키지화되어 부팅/구성 시간 단축
* AMI는 __특정 지역__ 을 위해 구축되며 여러 지역에 걸쳐 복사할 수 있습니다.
* 다음 위치에서 EC2 인스턴스를 시작할 수 있습니다.
    * __공용 AMI__: AWS 제공
    * __자신만의 AMI__: 직접 만들고 유지 관리합니다.
    * __AWS 마켓플레이스 AMI__: 다른 사람이 만든 AMI(판매 가능성 있음)

### 3.2. AMI Process (from an EC2 instance)
* Start an EC2 instance and customize it
* Stop the instance (for data integrity)
* Build an AMI – this will also create EBS snapshots
* Launch instances from other AMIs

* EC2 인스턴스 시작 및 사용자 지정
* 인스턴스 중지(데이터 무결성을 위해)
* AMI 구축 – EBS 스냅샷도 생성됩니다.
* 다른 AMI에서 인스턴스 시작

![Untitled](/assets/img/aws_saa_c02/20221229_ec2instancestorage_ami_process.JPG)


## 4. EC2 Instance Store
* EBS volumes are __network drives__ with good but “limited” performance
* __"If you need a high-performance hardware disk, use EC2 Instance Store"__

* Better I/O performance
* EC2 Instance Store lose their storage if they’re stopped (ephemeral)
* Good for buffer / cache / scratch data / temporary content 
* Risk of data loss if hardware fails
* Backups and Replication are your responsibility 

* EBS 볼륨은 양호하지만 "제한된" 성능을 가진 __네트워크 드라이브__ 입니다.
* __"고성능 하드웨어 디스크가 필요한 경우 EC2 Instance Store를 사용하십시오."__

* I/O 성능 향상
* EC2 Instance Store가 중지되면 스토리지가 손실됨(수명이 짧은, 단명하는 = short-lived)
* 버퍼/캐시/스크래치 데이터/임시 콘텐츠에 적합 
* 하드웨어 장애 시 데이터 손실 위험
* 백업 및 복제는 귀사의 책임입니다.


## 5. EBS Volume Types


### Reference
Stephane Maarek,【글로벌 Best】 AWS Certified Solutions Architect Associate 시험합격!, Udemy
 (https://www.udemy.com/course/best-aws-certified-solutions-architect-associate/)