想法是將Windows 11安裝到VHD中, 然後於BCD中添加VHD啟動選項, 那麼這Windows 11系統就需要管理單一檔案便可, 
可與原廠預裝的Windows 10系統共存. 這樣做法有什麼好處, 首先現今SSD的讀寫出度已快得驚人, 完全可無視以VHD作為系統安裝碟所帶來的效能折損.
又不用考慮壓縮及分割分區的問題, 再來是相當方便備份, 當安裝好系統, 完成各種補丁更新, 安裝了常用的軟件並完成各種設定後,
在這最佳的狀況下, 只要將這個VHD檔複製一份便完成備份, 之後怎樣修改系統, 甚至把系統弄壞了, 只需直刪除此VHD檔, 
以備份的VHD檔再重新複製一份便能極速還原.

以下是製作流程:
按Win+r, 輸入cmd, 同時按下Ctrl+Shift+Enter以系統管理員身份開啟終端機, 
開啟命令行磁碟管理工具:
```
diskpart
```
為Windows 11創建VHD:
```
create vdisk file=C:\VHDs\Win11Pro.vhdx maximum=60000 type=expandable
```
選用這個VHD:
```
select vdisk file=C:\VHDs\Win11Pro.vhdx
```
掛載這個VHD:
```
attach vdisk
```
初始化VHD設為GPT分區格式:
```
convert gpt
```
為這個VHD創建一主分割區:
```
create partition primary
```
快速格式化這個分區為NTFS檔案系統:
```
format fs=ntfs quick label=Win11Pro
```
分配磁碟代號V給這個分區:
```
assign letter=v
```
完成創建VHD步驟, 離開:
```
exit
```

去下載最新的Windows 11官方ISO映像檔, 然後點選這個映像檔右鍵選掛接(我這例子的磁碟代號是E:), 
先查看一下映像檔包含的Windows 11的那些版本及對應的索引號碼(我查出的Windows 11 Pro是3):
```
dism /get-wiminfo /wimfile:e:\sources\install.wim
```
透過dism指令將Windows 11 Pro安裝到VHD:
```
dism /apply-image /imageFile:e:\sources\install.wim /index:3 /applydir:v:\
```

為了於BCD中添加VHD啟動選項, 要將隱藏的ESP分區掛載到一磁碟代號(我這裡以P:作例子):
```
mountvol p: /s
```
最後執行以下指令為VHD創建UEFI啟動檔案並新增啟動選項到ESP分區中:
```
bcdboot v:\windows /s p: /f uefi
```
至此已完成將Windows 11 Pro安裝到VHD中, 將電腦重新開機, 便會進入啟動選單, 點擊首項便可進入Windows 11 Pro.
經過更新, 安裝軟件等過程後, 返回Windows 10, 將C:\VHDs\Win11Pro.vhdx複製一份作為備份.
重複以上步驟, 你更可添加更多的VHD啟動, 例如Windows 11 Lite, Windows 11 LTSC等, 打做隨時替換的Windows系統.

