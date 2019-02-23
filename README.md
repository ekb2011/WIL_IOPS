# IOPS

## IOPS 란
**아이옵스**(Input/Output Operations Per Second,  **IOPS**)는  [HDD](https://ko.wikipedia.org/wiki/HDD "HDD"),  [SSD](https://ko.wikipedia.org/wiki/SSD "SSD"),  [SAN](https://ko.wikipedia.org/wiki/SAN "SAN")  같은 컴퓨터 저장 장치를  [벤치마크](https://ko.wikipedia.org/wiki/%EB%B2%A4%EC%B9%98%EB%A7%88%ED%81%AC_(%EC%BB%B4%ED%93%A8%ED%8C%85) "벤치마크 (컴퓨팅)")하는 데 사용되는 성능 측정 단위다. IOPS는 보통  [인텔](https://ko.wikipedia.org/wiki/%EC%9D%B8%ED%85%94 "인텔")에서 제공하는  [Iometer](https://ko.wikipedia.org/w/index.php?title=Iometer&action=edit&redlink=1 "Iometer (없는 문서)")  같은 벤치마크 프로그램으로 측정된다.

IOPS 측정값은 벤치마크 프로그램에 따라 다르다. 구체적으로는  [임의 접근](https://ko.wikipedia.org/w/index.php?title=%EC%9E%84%EC%9D%98_%EC%A0%91%EA%B7%BC&action=edit&redlink=1 "임의 접근 (없는 문서)")과  [순차 접근](https://ko.wikipedia.org/wiki/%EC%88%9C%EC%B0%A8_%EC%A0%91%EA%B7%BC "순차 접근")  여부, 벤치마크 프로그램의  [쓰레드](https://ko.wikipedia.org/wiki/%EC%93%B0%EB%A0%88%EB%93%9C "쓰레드")  갯수와  [큐](https://ko.wikipedia.org/wiki/%ED%81%90_(%EC%9E%90%EB%A3%8C_%EA%B5%AC%EC%A1%B0) "큐 (자료 구조)")의 크기, 데이터  [블록](https://ko.wikipedia.org/wiki/%EB%B8%94%EB%A1%9D "블록")  크기, 읽기 명령과 쓰기 명령의 비중 등에 따라 달라지며, 이외에도 많은 변수들이 있다. 일반적으로는 종합 IOPS, 임의 접근 읽기(Random Access Read) IOPS, 임의 접근 쓰기(Random Access Write) IOPS, 순차 접근 읽기(Sequential Access Read) IOPS, 순차 접근 쓰기(Sequential Access Write) IOPS로 나누어 측정한다.
## IOPS 측정
### FIO 설치

  

```
$wget http://dl.fedoraproject.org/pub/epel/7/x86_64/Packages/e/epel-release-7-11.noarch.rpm

$rpm -ivh epel-release-7-11.noarch.rpm

$yum install fio
```

  

### IOPS 측정(block size: 4K / rw: worst slow / num jobs: 4)

 

```
$fio --name=iotest --ioengine=libaio --direct=1 --readwrite=randwrite --numjobs=4 --iodepth=64 --group_reporting --filename=/root/iotest --size=10G --bs=4k
```



# LVM

## LVM 정의

* 파일 시스템이 LVM이 만든 가상의 블록 장치에 읽고/쓰기를하게 된다.  
  
![](http://img1.daumcdn.net/thumb/R1920x0/?fname=http%3A%2F%2Fcfile8.uf.tistory.com%2Fimage%2F2503D43D54B95EA712F097)

* 물리적 스토리지 이상의 추상적 레이어를 생성해서 논리적 스토리지(가상의 블록 장치)를 생성할 수 있게 해준다.

  

* 직접적으로 물리적 스토리지를 사용하는 것보다 다양한 측면에서 유연성을 제공한다.

 * 크기 조정 가능한 스토리지 풀(Pool)

* 온라인 데이터 재배치

* 편의에 따라 장치 이름 지정

* 디스크 스트라이핑

* 미러 볼륨

* 볼륨 스냅샷

  

### LVM2의 개선된 사항
  

* 유연한 용량

* 보다 효율적인 메타데이터 스토리지

* 보다 향상된 복구 포멧

* 새로운 ASCII 메타데이터 포멧

* 메타데이터로 원자 변경

* 메타데이터의 이중 복사본

  

* 스냅샷과 클러스터 지원을 제외하고 LVM1과 역 호환성이 있다.

  

* vgconvert 명령어로 LVM1 포멧에서 LVM2 포멧으로 변환이 가능하다.




## LVM 구성법

### 1. Block Storage 설정 (500G * 2)

  

### 2. fdisk 활용 Partition 생성, System ID Linux LVM으로 변경(vdb, vdc 각각)

  
```
$fdisk /dev/vdb + n + p

(Partition number: 1 / First cylinder: 204800 / Last cylinder: maximum)
```
  

### 3. System ID Linux LVM으로 변경

```
+ t + 8e + w
```
  

### 4. Physical Volume 생성
```
$yum install lvm2 -y

$pvcreate /dev/vdb1

$pvcreate /dev/vdc1
```
  

### 5. Virtual Group 생성
```
$vgcreate vg001 /dev/vdb1 /dev/vdc1
```
  

### 6. Logical Volume 생성
```
$lvcreate -i 2 -L 500GB -n lv001 vg001
```
  

### 7. Logical Volume 포맷
```
$mkfs.ext4 /dev/vg001/lv001
```
  

### 8. LVM Mount
```
$mkdir /data

$chmod 755 /data

$mount /dev/vg001/lv001 /data
```
  

### 9. Mount 상황 확인
```
$df -l
```