# 2024-Arduino-Yun
方法1：yun透過網路一個一個下載檔案
mkdir -p /mnt/sda1/packages #創建更新檔的資料夾
cd /mnt/sda1/packages #進入資料夾
#下載檔案
wget --no-check-certificate http://downloads.arduino.cc/openwrtyun/1/packages/Packages 
wget --no-check-certificate http://downloads.arduino.cc/openwrtyun/1/packages/Packages.sig
wget --no-check-certificate http://downloads.arduino.cc/openwrtyun/1/packages/Packages.gz
cat Packages | grep "Filename: " >downloadfile.txt
sed -i 's/Filename: /http:\/\/downloads.arduino.cc\/openwrtyun\/1\/packages\//g' downloadfile.txt
wget --no-check-certificate -i downloadfile.txt
約 20 分鐘後，檔案將下載到 Micro SD 卡。

方法2：下載壓縮檔再解壓縮
1.	Yun透過網路下載openwrtyun.tar.gz
wget -O openwrtyun.tar.gz https://www.dropbox.com/s/ilfb7hainhq9r0o/openwrtyun.tar.gz?dl=0 --no-check-certificate
md5sum openwrtyun.tar.gz	
#確保 md5 哈希為f4351fcf4ebeda3eec702fa98812cf96
#(回傳) f4351fcf4ebeda3eec702fa98812cf96  openwrtyun.tar.gz
tar -xvzf openwrtyun.tar.gz

2.	先下載好並解壓縮再放到SD卡
前往網頁並下載檔：https://www.dropbox.com/s/ilfb7hainhq9r0o/openwrtyun.tar.gz?dl=0
SD卡創建資料夾openwrtyun
解壓縮後放入SD卡openwrtyun資料夾內

更改opkg update指令方法
1.	透過ssh更改：
nano /etc/opkg.conf  #編輯opkg.conf
#src/gz attitude_adjustment http://downloads.arduino.cc/openwrtyun/1/packages	#將原本指令註解
src/gz local file:////mnt/sda1/(創建的資料夾名稱)	#新增指令，添加剛才創建的資料夾路徑
#ctrl+x, y, enter退出編輯
opkg update #執行更新

2.	透過Web UI更改：
  
      

資料來源：https://forum.arduino.cc/t/setup-local-packages-repository/230811
檔案連結：
		https://downloads.arduino.cc/openwrtyun/1.6.2/packages/index.html?_gl=1*12lgdr0*_gcl_au*MTAwMTc2MDk2MC4xNzE4ODgzMjEw*FPAU*MTAwMTc2MDk2MC4xNzE4ODgzMjEw*_ga*MTYzMzA0NDA0Ni4xNzE4ODgzMjA4*_ga_NEXN8H46L5*MTcyMDUzNTAxOC40My4xLjE3MjA1NDIzMzQuMC4wLjg4NzE3OTM2OQ..*_fplc*aDgzeEFVZVdjcTZSd0VROWQlMkJ5SGNuMmdEZDJVRGM1YmNMNXplTzRCaHRKZEVpdTFLeG9jNnNzNFgyWU1WeUs1RTBKT0NyUGQyWEtVaGVxQlhnUTJCUnclMkIxdXFMYmJYOGFjTEdhUUdlb0dxSXZvck5odWhNVFRwUENIR09BUSUzRCUzRA..
	https://www.dropbox.com/scl/fi/9f23nb5sbod98ude7rbj8/openwrtyun.tar.gz?e=2&file_subpath=%2Fopenwrtyun&rlkey=i0v7xtw5livfu5h6b6m36cpb7&dl=0
https://docs.arduino.cc/tutorials/yun-rev2/yun-sys-restore/
