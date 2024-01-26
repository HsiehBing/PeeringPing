# VPC peering test

最後編輯時間：2023.6.17

## 建置設定

1. VPC建置

說明：

- 可以選擇VPC only介面，這個介面比較是用來在一個AZ中建立多個subnet使用。

如果選擇VPC and more 可以一次建立Subnet、Route tables、Network connection(IGW、NAT gateway)

- 選擇IPv4的CIDR可以有三個大方向選：10.0.0.0/8、172.16.0.0/12、192.168.0.0/16 可以根據需求選擇
- 選擇AZ，以及public以及private subnet數量
- NAT gateway可以用來將資料由private 單向傳出
- DNS options 預設開啟

需要至兩個region都設定VPC

額外說明：如果使用private subnet 可以由NAT Gateway將資料傳出，但是如果要造放需要透過跳板機、SSM等方法才能登入，跳板機的使用是另外在public subnet開一台EC2，先ssh到跳板機再ssh到private subnet。

1. Peering connection 設定
- 至vpc peering connectio設定，設定名稱以及選擇剛剛設定的VPC，以及預對街的VPC，設定完後至另一個Region的VPC中點選確認。
- 至該VPC 的 private subnet 的Route Table加上 peering的範圍以及peering，加上後即可連線。

### VPC Route table 設定

1.在public subnet與private subnet 各需要一張route table。在public subnet需要用IGW、private NAT gateway

|  | EC2 Name | VPC Name | VPC range |
| --- | --- | --- | --- |
| Singapore | Bing-test-si | Bing-vpc | 10.0.0.0/16 |
| Tokyo | Bing-peering-north | Bing-northeast-vpc | 172.31.0.0/16 |

|  | Public IP | Private IP |
| --- | --- | --- |
| Singapore | 54.179.29.64 | 10.0.14.252 |
| Tokyo | 18.183.255.132 | 172.31.0.128 |

## 連線資訊

**Singapore**

**Public:54.179.29.64**

**Private:10.0.14.252**

連線

ssh -i "Bing-Singapore.pem" ec2-user@ec2-54-179-29-64.ap-southeast-1.compute.amazonaws.com

**Tokyo**

**Public:18.183.255.132**

**Private:172.31.0.128**

連線

ssh -i "Bing-Key.pem" ec2-user@ec2-18-183-255-132.ap-northeast-1.compute.amazonaws.com

## 安裝套件

sudo yum install cronie -y

sudo service crond start

sudo yum install tree -y

## 操作設定

### **Singapore**

**Public**

mtr:

/usr/sbin/mtr -i 5 -c 60 -r 18.183.255.132  &> /home/ec2-user/MTRTest/public/S2TPublic_$(date +%m-%d_%H:%M).txt

crontab:

sh /home/ec2-user/MTRTest/public/S2Tpublic.sh

ping:

ping -i 60 18.183.255.132 2>&1 | while IFS= read -r line; do echo "$(date +"%m-%d %H:%M") $line"; done >>S2Tpublic.txt &

**Private**

/usr/sbin/mtr -i 5 -c 60 -r 172.31.0.128  &> /home/ec2-user/MTRTest/private/S2TPrivate_$(date +%m-%d_%H:%M).txt

crontab

sh /home/ec2-user/MTRTest/public/S2Tprivate.sh

ping

ping -i 60 172.31.0.128 2>&1 | while IFS= read -r line; do echo "$(date +"%m-%d %H:%M") $line"; done >>S3private.txt &

### **Tokyo**

**Public**

mtr:

/usr/sbin/mtr -i 5 -c 60 -r  54.179.29.64  &>  /home/ec2-user/MTRTest/public/T2SPublic_$(date +%m-%d_%H:%M).txt

crontab:

sh /home/ec2-user/MTRTest/public/T2Spublic.sh

ping:

ping -i 60 54.179.29.64 2>&1 | while IFS= read -r line; do echo "$(date +"%m-%d %H:%M") $line"; done >>T2Spublic.txt &

**Private**

mtr:

/usr/sbin/mtr -i 5 -c 60 -r  10.0.14.252  &>  /home/ec2-user/MTRTest/private/T2SPrivate_$(date +%m-%d_%H:%M).txt

crontab:

sh /home/ec2-user/MTRTest/public/T2Sprivate.sh

ping:

ping -i 60 10.0.14.252 2>&1 | while IFS= read -r line; do echo "$(date +"%m-%d %H:%M") $line"; done >>T2Sprivate.txt &

## 資料下載

### Ping 資料

scp -i "Bing-Singapore.pem" ec2-user@ec2-54-179-29-64.ap-southeast-1.compute.amazonaws.com:/home/ec2-user/S2Tpublic.txt /Users/bing.hsieh/Desktop/KronosTest/S2Tpublic.txt

scp -i "Bing-Singapore.pem" ec2-user@ec2-54-179-29-64.ap-southeast-1.compute.amazonaws.com:/home/ec2-user/S3private.txt /Users/bing.hsieh/Desktop/KronosTest/S2Tprivate.txt

scp -i "Bing-Key.pem" ec2-user@ec2-18-183-255-132.ap-northeast-1.compute.amazonaws.com:/home/ec2-user/T2Spublic.txt /Users/bing.hsieh/Desktop/KronosTest/T2Spublic.txt

scp -i "Bing-Key.pem" ec2-user@ec2-18-183-255-132.ap-northeast-1.compute.amazonaws.com:/home/ec2-user/T2Sprivate.txt /Users/bing.hsieh/Desktop/KronosTest/T2Sprivate.txt

### Mtr資料

scp -i "Bing-Singapore.pem" ec2-user@ec2-54-179-29-64.ap-southeast-1.compute.amazonaws.com:/home/ec2-user/MTRTest/public /Users/bing.hsieh/Desktop/KronosTest/Mtr_S2Tpublic

scp -i "Bing-Singapore.pem" ec2-user@ec2-54-179-29-64.ap-southeast-1.compute.amazonaws.com:/home/ec2-user/MTRTest/private /Users/bing.hsieh/Desktop/KronosTest/Mtr_S2Tprivate

scp -i "Bing-Key.pem" ec2-user@ec2-18-183-255-132.ap-northeast-1.compute.amazonaws.com:/home/ec2-user/MTRTest/public /Users/bing.hsieh/Desktop/KronosTest/Mtr_T2Spublic

scp -i "Bing-Key.pem" ec2-user@ec2-18-183-255-132.ap-northeast-1.compute.amazonaws.com:/home/ec2-user/MTRTest/private /Users/bing.hsieh/Desktop/KronosTest/Mtr_T2Sprivate
