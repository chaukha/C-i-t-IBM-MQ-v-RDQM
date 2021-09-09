Cài IBM MQ 9.2 và RDQM trên RHEL 8.2

- Mô hình: 3 Server RHEL, mỗi Server có 1 IP, 2 Disk 

I. Cài đặt IBM MQ 9.2
1. Accept license | Nếu dã từng cài MQ trên máy thì bắt buộc chạy lệnh crtmqpkg
Cài đặt các gói : yum install -y MQSeriesRuntime-9.2.0-0.x86_64.rpm MQSeriesJRE-9.2.0-0.x86_64.rpm MQSeriesSDK-9.2.0-0.x86_64.rpm MQSeriesServer-9.2.0-0.x86_64.rpm MQSeriesClient-9.2.0-0.x86_64.rpm MQSeriesExplorer-9.2.0-0.x86_64.rpm MQSeriesGSKit-9.2.0-0.x86_64.rpm MQSeriesSamples-9.2.0-0.x86_64.rpm MQSeriesSamples-9.2.0-0.x86_64.rpm
1. Trên một server có thể cài nhiều instance của MQ, do vậy MQ cho phép chọn một instance là primary installation -> /opt/mqm/bin/setmqinst -i -p /opt/mqm
2. Set môi trường ->  . /opt/mqm/bin/setmqenv -s  | dspmqver
II. Cài đặt RDQM
1. Cài đặt Kernel
    1. Add sign: -> rpm --import https://packages.linbit.com/package-signing-pubkey.asc
    2. Cài đặt kernel -> yum install Advanced/RDQM/PreReqs/el8/kmod-drbd-9/kmod-drbd-9.0.23_4.18.0_193-1.x86_64.rpm
2. Cài đặt DRBD: -> yum install -y Advanced/RDQM/PreReqs/el8/drbd-utils-9/*
3. Cài đặt Pacemaker2: -> yum install -y Advanced/RDQM/PreReqs/el8/pacemaker-2/*
4. Cài đặt drbdpool: kiểm tra lại server đã nhận disk chưa: -> lsblk
    1. Cài đặt lvm2: -> yum install -y vlm2
    2. Tạo Physical Volume: pvcreate /dev/sdb
    3. Tạo volume group : vgcreate drbdpool /dev/sdb
    4. grant sudo access: -> nano /etc/sudoers.d/mqm -> mqm ALL=(root) NOPASSWD: /opt/mqm/bin/crtmqm, /opt/mqm/bin/dltmqm, /opt/mqm/bin/rdqmadm, /opt/mqm/bin/rdqmstatus
5.  Setup passwordless SSH
- Đặt password chung user mqm cho cả 3 server -> passwd mqm
- -> usermod -d /home/mqm mqm
- -> mkhomedir_helper mqm
- -> su mqm
- -> ssh-keygen -t rsa -f /home/mqm/.ssh/id_rsa -N ''
- -> ssh-copy-id -i /home/mqm/.ssh/id_rsa.pub <private IP address of node>
-  -> passwd -d mqm
	-> passwd -l mqm
6. Cấu hình HA Group: -> nano /var/mqm/rdqm.ini
    1. Template đối với RDQM của MQ 9.1 trở về trước: 
Node:
  HA_Replication=
Node:
  HA_Replication=
Node:
  HA_Replication=
	2. Template đối với RDQM của 9.2 trở về sau:

Node:
  Name=
  HA_Replication=
Node:
  Name=
  HA_Replication=
Node:
  Name=
  HA_Replication=
7. Sync -> rdqmadm -c

8. Tạo 1 RDQM -> crtmqm -p 1414 -sx RDQM1

https://github.com/ibm-messaging/mq-rdqm/blob/master/cloud/aws/README.md

9. Kiểm tra status: rdqmstatus –m qmgrName
Nâng cao, tạo floating IP: https://www.royalcyber.com/blog/middleware/high-availability-of-replicated-data-queue-manager/
