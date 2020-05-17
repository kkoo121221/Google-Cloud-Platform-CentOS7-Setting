# 【目標】
在 Google Cloud Platform (GCP) 平台上建立的 CentOS 7 作業系統環境下，安裝 xrdp + gnome 提供圖形化 (GUI) 介面，並安裝 google-drive-ocamlfuse 與 qbittorrent，達成與在 Windows 環境下安裝 Google Drive Stream (Google 雲端檔案串流) + qbittorrent 一致的效果與功能。

# 【步驟】
## 1. 安裝圖形介面 (xrdp + gnome) 並開始運作
進入GCP SSH主控台後
[指令] sudo yum -y groups install "GNOME Desktop"
(安裝gnome)
[指令] sudo rpm -Uvh http://download.fedoraproject.org/pub/epel/epel-release-latest-7.noarch.rpm
(確定epel REPO有無安裝，在一般狀態下GCP提供的CentOS 7都已經安裝了)
[指令] yum install xrdp tigervnc-server
(安裝vnc + xrdp)
[指令] systemctl enable xrdp.service
(設定開機自啟動)
[指令] systemctl start xrdp.service
(開啟xrdp)
[指令] netstat -antup | grep xrdp
(查看使用的Port，預設是使用與Windows遠端桌面一致的3389，需修改可使用指令「sudo nano /etc/xrdp/xrdp.ini」)
[指令] firewall-cmd --state
(確認防火牆狀態)
[指令] firewall-cmd --list-all --zone=public
(查看防火牆開放的Port)
[指令] firewall-cmd --permanent --zone=public --add-port=3389/tcp
[指令] firewall-cmd --reload
(開放3389 Port並讓防火牆重啟以套用新設定)

## 2. 登入圖形介面
(1) 修改帳號的密碼以順利登入
開啟GCP SSH主控台
[指令] sudo passwd [目前用戶名稱]
(修改目前登入帳號的密碼)
[指令] sudo passwd root
(修改root帳戶的密碼)
(2) 開啟Windows的「遠端桌面連線」，輸入VM的公開IP連線
(3) 出現「未知的授權」，按「是」繼續。
(4) 進入xrdp登入頁，先使用root帳號登入。

## 3. 初始設定
(1) 進入 應用 ---> Setting ---> Language & Region
設定格式與語言為 漢語(台灣)
(2) 切換到 Time 分頁
切換時區 +8 Taipei
(3) 拉到最底下的關於分頁進入
進入使用者分頁
設定自己主要一般帳號語言為「漢語(台灣)」
將自己的主要一般帳號權限設為管理員
(4) 重啟系統

## 4. 安裝 google-drive-ocamlfuse 並掛載 Google Drive
### (1) 使用遠端桌面連線登入自己的帳號後，開啟終端機
[指令] sudo yum update -y
[指令] sudo yum install -y git
[指令] sudo yum install -y hg
[指令] sudo yum install -y sqlite-devel fuse fuse-devel libcurl-devel zlib-devel m4
(必要元件確認與安裝)
[指令] sudo yum install -y ocaml ocamldoc ocaml-camlp4-devel
[指令] sudo wget https://raw.github.com/ocaml/opam/master/shell/opam_installer.sh -O - | sh -s /usr/local/bin/
[指令] sudo yum install
[指令] opam init
(安裝opam)

[指令] opam install google-drive-ocamlfuse
[指令] export DISPLAY=:0.0
[指令] source ~/.bash_profile
(安裝google-drive-ocamlfuse)
[指令] opam list
(確認google-drive-ocamlfuse有無安裝完成)

## ▼遇到過的錯誤
1 - 出現缺少必要組建 patch 的錯誤，無法安裝

[指令] sudo yum install -y patch
安裝patch完後重新執行opam安裝指令

2 - 因無法安裝 conf-gmp，google-drive-ocamlfuse 安裝失敗
自己遇到是因為沒有 gmp-devel 組件造成無法安裝 conf-gmp，主要還是要看下 TroubleShooting 跟錯誤Log，不過應該大部分GCP的是這個錯誤 ?

[指令] sudo yum install -y http://mirror.centos.org/centos/7/os/x86_64/Packages/gmp-devel-6.0.0-15.el7.x86_64.rpm
(或是「sudo yum install -y gmp-devel」應該也行)

## 5. 設定google-drive-ocamlfuse
開啟終端機
[指令] google-drive-ocamlfuse
瀏覽器跳出，登入你想要掛載雲端硬碟的帳號

## ▼如果是要掛載那支帳號所建的小組雲端硬碟 (Google 教育版/G Suite)
(1) 先建立一個label
google-drive-ocamlfuse -label [名稱自取，記得就好]
(2) 複製你要掛載的小組雲端硬碟 folder ID
打開雲端硬碟首頁，切到小組雲端硬碟分頁，打開你要掛載的小組雲端硬碟
此時的網址
https://drive.google.com/drive/u/0/folders/0ABo0QwXXXXXXXXXX
黃色框框紅色字的就是你要複製過去的ID
(3) 開啟那個label的設定檔
[指令] sudo nano ~/.gdfuse/<你取的label的名稱>/config
(4) 找到「team_drive_id」這個欄位，把ID貼過去
像這樣
team_drive_id=0ABo0QwXXXXXXXXXX
(5) 按y再按Enter儲存設定

## ▼遇到過的錯誤
1 - Can't open display: 0.0

關閉連線，重新連線，登入root帳號，打開終端機
[指令] xhost +
執行完後重開機

## 6. 掛載Google Drive
開啟終端機
[指令] mkdir /mnt/Files/GDrive
(新建一個資料夾「GDrive」在 /mnt/Files 中供掛載用，當然也能夠自訂路徑跟自訂資料夾名稱)
[指ㄧ令] google-drive-ocamlfuse /mnt/Files/GDrive
(掛載 Google Drive)
* 如果是要掛小組雲端硬碟，是使用「google-drive-ocamlfuse -label [你取的名稱] /mnt/Files/GDrive」

## 7. 安裝qbittorrent
[指令] sudo yum install -y qbittorrent

# 【目前還在解決的問題】
## 1. qbittorrent 檔案下載完成後無法做種 (按強制繼續或繼續，做種幾秒鐘後跳回已完成)
爬了文可能是非固定IP的問題，不過真的很迷，之後測試
## 2. qbittorrent Linux 版本沒有提供選分類自動設定路徑達成自動管理
這個就真沒辦法了，只能一次一次設定，或是等自己功力夠後自己改良吧orz

# 【參考/引用資料】
1. How to install Desktop Environments on CentOS 7? - Unix & Linux Stack Exchange
https://unix.stackexchange.com/questions/181503/how-to-install-desktop-environments-on-centos-7
2. XYZ的筆記本 - CentOS 7 安裝遠端桌面
https://xyz.cinc.biz/2016/03/linux-gui-xrdp.html
3. centos挂载google-drive - 南思工作室
https://www.inansi.com/2018/05/24/centosgoogle-drive/
4. gmp-devel-6.0.0-15.el7.x86_64.rpm CentOS 7 Download
https://centos.pkgs.org/7/centos-x86_64/gmp-devel-6.0.0-15.el7.x86_64.rpm.html
5. Mount Google Drive On Linux With Google Drive Ocamlfuse Client
https://www.2daygeek.com/mount-access-google-drive-on-linux-with-google-drive-ocamlfuse-client/
6. Team Drives · astrada/google-drive-ocamlfuse Wiki · GitHub
https://github.com/astrada/google-drive-ocamlfuse/wiki/Team-Drives
7. 转载 本机运行x程序出现：Can't open display 原因及其解决方法
https://blog.csdn.net/wuyao721/article/details/3678859
8. CentOS 圖片
http://www.weithenn.org/2017/10/centos-74-journey-index.html