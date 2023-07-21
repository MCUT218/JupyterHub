# JupyterHubGPU
一個帶有 JupyterLab、Tensorflow 和 GPU 支援的 Docker 化 JupyterHub 實例

本說明檔將涵蓋複製所提供的配置所需的步驟，以及在需要時如何進一步自訂配置。

此環境並不適用於生產，因為沒有強大的安全措施 - 請自行承擔使用風險。

## 功能
多用戶 JupyterLab 介面，支援 Tensorflow 和 GPU
Hub 在 Docker 容器中運行
每個使用者的筆記本（Lab）都在專用的 Docker 容器中啟動
共享資料夾（例如使用者的家目錄和資料夾）被掛載到使用者的 Docker 容器，以保持資料的持久性
預先需求
這些套件必須安裝在主機上（目前運行 Ubuntu 20.04 LTS），套件版本表示目前已安裝的版本：

Nvidia Cuda 驅動程式（目前為 nvidia-driver-535，由 Ubuntu "Additional Drivers" 應用程式安裝）
docker（20.10.7）
docker-compose（1.25.0）
nvidia-docker2（2.6.0）
初步步驟
在主機上創建一個新使用者（例如 jupyterhub），用於管理 JupyterHub 和存放 Hub 的配置文件。
將您的使用者加入 docker 群組，以便在不使用 sudo 的情況下執行 docker 命令。
通過運行 docker run --rm --gpus all nvidia/cuda:10.1-base nvidia-smi 測試 nvidia-docker 中的 GPU 是否工作。
在主機上配置必要的共享資料夾掛載，例如 /home/jh_users/ 用於存放使用者的家目錄（我們將其與 UNIX 主機使用者分開），以及 /mnt/sdb2/ 用於共享資料（數據集、模型等）。
配置設置環境
環境構建為 docker-compose 應用程式。以下是複製配置的步驟。
複製這個存儲庫或者創建一個包含設置環境的文件夾（例如 /home/jupyterhub/JupyterHubGPU/，從現在開始將其指定為根目錄）。

## 配置 Hub 映像
./jupyterhub/ 包含 Hub 配置和映像。
./jupyterhub/Dockerfile 定義了 Hub 映像。我們在這裡創建了默認的管理員 jhadmin，並設置了默認密碼。這個使用者僅存在於 Hub 容器中，可以從 Hub 控制面板中添加新使用者。我們還將工作目錄（WORKDIR）設置為 /srv/jupyterhub，並將默認命令設置為運行 Hub 服務器的 jupyterhub 命令。
./jupyterhub/jupyterhub_config.py 描述了 JupyterHub 服務器應用程式的配置。有關詳細信息，請參閱以下各節。
此映像的構建將由 docker-compose 完成，因此我們現在不需要手動構建它。

## 運行 Hub
首次運行時，轉到根目錄並運行：
docker-compose up -d
可選的 -d 參數將 Hub 作為服務運行。
第一次運行此命令時，將構建 jupyterhub 映像並在 <host-ip>:8000 上運行服務器。默認登錄帳號為：jhadmin，密碼在 ./jupyterhub/Dockerfile 中設置。
警告：一旦您第一次運行容器，或者每次更新配置後，請立即將管理員密碼更改為安全密碼！要這樣做，運行 docker exec -it jupyterhub-container /bin/bash，然後運行 passwd jhadmin 選擇新密碼。

下次可以使用 docker-compose start 和 docker-compose stop 啟動/停止容器（例如在更新 jupyterhub_config.py 時）。

要停止並移除容器，請使用 docker-compose down。

## 添加新使用者
在主機上運行 docker exec -it jupyterhub-container /bin/bash。
運行 adduser <username>。
運行 passwd <username> 並設置密碼。
在 Web 介面中以管理員身份登錄，在管理員菜單（頂部欄）中添加新使用者 <username>。
添加新的自定義 Docker 映像
要添加新的映像，例如凍結特定版本的 tensorflow，只需複製 tensorflow-notebook 的結構，然後相應編輯 Dockerfile。然後，像在上一節中所做的那樣構建映像並將映像添加到 jupyterhub_config.py 中的列表中。

## 使用者環境
使用者可以使用 JupyterLab 的實例，並擁有以下文件夾結構：

./work：啟用了持久性的工作目錄，它保存在主機機器上的 /home/jh_users/{username}/ 中。
./SHARED：共享的數據文件夾以只讀方式掛載，它保存在主機機器上的 /mnt/sda2/ 中。

