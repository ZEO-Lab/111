﻿# 2024-Arduino-Yun
## 前言
Arduino Yun自從2014年發售，距今已經超過10年(2024)。官方早已經停止更新相關，國內國外也不再討論此控制板，有些指令功能也產生錯誤。於是本人把所有尋到的解法以及資料放在此處，希望能夠解決一些的問題。
> [!WARNING]
> 請具備相關知識與技術再進行操作

## 更新OpenWrt v1.5.3
[Arduino官網](https://www.arduino.cc/en/software)已經無法下載Yun相關的軟體。但使用者可以在[Restoring the 1.5.3 Image on Arduino Yún](https://docs.arduino.cc/tutorials/yun-rev2/yun-sys-restore/)或者從[資料夾](YunSysupgradeImage_v1.5.3)下載 v1.5.3版本 的檔案，放入Yun的SD卡。接著請參照[官方教學](https://docs.arduino.cc/tutorials/yun-rev2/yun-sys-upgrade/)，按照其步驟進行更新，[^3]

[^3]: [Restoring the 1.5.3 Image on Arduino Yún](https://docs.arduino.cc/tutorials/yun-rev2/yun-sys-restore/) 

## 沒有偵測到SD卡
此方法主要針對硬體的問題，且常發生在使用一段時間的板子上。  

此圖為Yun背後的SD槽，其中有紅色框框內是由兩個金屬彈簧組成的開關，一旦插入SD卡則金屬彈簧接觸短路，系統會讀取SD卡內容。 
但時間久了，當彈簧彈性疲乏後會導致開關無法閉合，導致系統會以為沒有插入SD卡而不會讀取。[^5]   

<img src = "doc/金屬彈簧片.jpg" width = "50%">  
  
  
解決方法很簡單，建議使用絕緣工具，把金屬彈簧稍微凹回來，確保插入SD卡時開關會閉合即可。[^6]

> [!CAUTION]
> 矯正前，請先關閉電源，並使用絕緣工具矯正  

> [!NOTE]
> 若還是沒有成功，可以參考下一段文章，格式化SD卡。

## Expand Disk Space
Yun系統內部空間極小，需要透過SD來存放檔案，因此Arduino釋出[官方教學](https://docs.arduino.cc/tutorials/yun-rev2/expanding-yun-disk-space/)，只要燒錄官方程式碼後再進行一連串確認，就能將SD劃出一部分空間給系統[^4]。  

> [!TIP]
> SD卡必須清空，建議使用SD Formatter[^7]對SD卡格式化，確保你已備份資料。

問題排解： 
* ```The micro SD card is not available```：可以輸入指令```df -k```確認系統是否偵測到SD卡。可以重新格式化SD卡，或者閱讀上一篇文章來排除問題。  
* ```err. with opkg, check internet connection``` ：
此錯誤為執行 ```opkg update```失敗時所產生，若你已經按照下篇文章進行更新，系統會自動從SD內部尋找更新檔，雖然能夠解決此問題，但這樣會衍生下一個問題。
* ```err. formatting to FAT32``` ：此錯誤為執行```mkfs.vfat /dev/sda1 #在SD建立FAT文件系統```失敗所產生。  
為了執行```opkg update```，SD卡必須存放著更新檔，但使系統無法建立FAT文件系統。於是我們需要一個隨身碟來儲存更新檔，把SD卡空出來。  
幸好Yun有一個USB插座，輸入```df -k```取得隨身碟所在位置(sdb1)，接著修改 opkg.conf 裡的指令，使其改抓取隨身碟的更新檔。
這樣不僅能讓SD完全格式化，也能透過隨身碟取得更新檔。  
擴展成功後，只要把更新檔重新下載到SD卡即可。

## opkg update
在原本更新Yun OpenWrt系統的opkg時會產生以下錯誤[^2]：
```
Downloading http://downloads.arduino.cc/openwrtyun/1/packages/Packages.gz.  
Downloading http://downloads.arduino.cc/openwrtyun/1/packages/Packages.sig.
Signature check failed.
Remove wrong Signature file.
Collected errors:
* opkg_download: Failed to download http://downloads.arduino.cc/openwrtyun/1/packages/Packages.gz: Error.
* opkg_download: Failed to download http://downloads.arduino.cc/openwrtyun/1/packages/Packages.sig: Error.
```
其中有兩個錯誤，其中一個是`Signature check failed`，另一個則是`Failed to download`。  
對於`Failed to download`，參考相關資料後[^1]後我總結出以下兩種方式來下載更新檔

1. yun透過網路一個一個下載檔案，約 20 分鐘後，檔案將下載到 SD 卡。 [^1]
    ```
    mkdir -p /mnt/sda1/packages   #創建更新檔的資料夾  
    cd /mnt/sda1/packages         #進入資料夾

    #下載檔案
    wget --no-check-certificate http://downloads.arduino.cc/openwrtyun/1/packages/Packages   
    wget --no-check-certificate http://downloads.arduino.cc/openwrtyun/1/packages/Packages.sig
    wget --no-check-certificate http://downloads.arduino.cc/openwrtyun/1/packages/Packages.gz
    cat Packages | grep "Filename: " >downloadfile.txt
    sed -i 's/Filename: /http:\/\/downloads.arduino.cc\/openwrtyun\/1\/packages\//g' downloadfile.txt
    wget --no-check-certificate -i downloadfile.txt
    ```
2. 下載壓縮檔再解壓縮
    1.	Yun透過網路下載openwrtyun.tar.gz
        ```
        wget -O openwrtyun.tar.gz https://www.dropbox.com/s/ilfb7hainhq9r0o/openwrtyun.tar.gz?dl=0 --no-check-certificate #下載檔案
        md5sum openwrtyun.tar.gz #確保 md5 哈希為f4351fcf4ebeda3eec702fa98812cf96
        #(回傳) f4351fcf4ebeda3eec702fa98812cf96  openwrtyun.tar.gz
        tar -xvzf openwrtyun.tar.gz #解壓縮檔案
        ```
    2.	先下載好並解壓縮再放到SD卡：  
        * 前往網頁並下載檔：https://www.dropbox.com/s/ilfb7hainhq9r0o/openwrtyun.tar.gz?dl=0  
        * SD卡創建資料夾openwrtyun  
        * 解壓縮後放入SD卡 openwrtyun 資料夾內

接著要修改opkg update指內容，也要解決`Signature check failed`。
1.	透過ssh更改：  
    ```
    nano /etc/opkg.conf  #編輯opkg.conf 
    ```
    在opkg.conf裡
    ```
    #src/gz attitude_adjustment http://downloads.arduino.cc/openwrtyun/1/packages	#將原本指令註解
    src/gz local file:////mnt/sda1(更新檔所在記憶體)/packages(SD更新檔的資料夾名稱)
    #新增指令：添加SD卡存放更新檔的資料夾路徑
    #option check_signature #將這行註解解決 Signature check failed
    ```
    #ctrl+x, y, enter退出編輯
    ```
    opkg update #執行更新
    ```  
2.	透過Web UI更改：  
    <img src = "doc/opkg-update.gif" width = "100%"> 
      
檔案連結：
* [OpenWrtYun v1.6.2](https://downloads.arduino.cc/openwrtyun/1.6.2/packages/index.html?_gl=1*12lgdr0*_gcl_au*MTAwMTc2MDk2MC4xNzE4ODgzMjEw*FPAU*MTAwMTc2MDk2MC4xNzE4ODgzMjEw*_ga*MTYzMzA0NDA0Ni4xNzE4ODgzMjA4*_ga_NEXN8H46L5*MTcyMDUzNTAxOC40My4xLjE3MjA1NDIzMzQuMC4wLjg4NzE3OTM2OQ..*_fplc*aDgzeEFVZVdjcTZSd0VROWQlMkJ5SGNuMmdEZDJVRGM1YmNMNXplTzRCaHRKZEVpdTFLeG9jNnNzNFgyWU1WeUs1RTBKT0NyUGQyWEtVaGVxQlhnUTJCUnclMkIxdXFMYmJYOGFjTEdhUUdlb0dxSXZvck5odWhNVFRwUENIR09BUSUzRCUzRA..)
* [opkg update file](https://www.dropbox.com/scl/fi/9f23nb5sbod98ude7rbj8/openwrtyun.tar.gz?e=2&file_subpath=%2Fopenwrtyun&rlkey=i0v7xtw5livfu5h6b6m36cpb7&dl=0)


[^1]: [Setup local packages repository](https://forum.arduino.cc/t/setup-local-packages-repository/230811)  
[^2]: [opkg update failed on Arduino Yun](https://forum.arduino.cc/t/opkg-update-failed-on-arduino-yun/276539)  
[^3]: [Restoring the 1.5.3 Image on Arduino Yún](https://docs.arduino.cc/tutorials/yun-rev2/yun-sys-restore/)  
[^4]: [Expanding Yún Disk Space](https://docs.arduino.cc/tutorials/yun-rev2/expanding-yun-disk-space/)  
[^5]: [SD card not present in Yun](https://forum.arduino.cc/t/sd-card-not-present-in-yun/289790)  
[^6]: [Arduino YUN and SD Card: doesn't work!](https://forum.arduino.cc/t/arduino-yun-and-sd-card-doesnt-work/241757/16)  
[^7]: [SD Memory Card Formatter for Windows/Mac](https://www.sdcard.org/downloads/formatter/)




