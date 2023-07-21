# JupyterHubGPU
一個帶有登入系統的 JupyterLab、Tensorflow 和 Nvidia GPU 支援的 Docker容器  

本說明檔將涵蓋複製所提供的配置所需的步驟，以及在需要時如何進一步自訂配置。  


## 功能
多用戶 JupyterLab 介面，支援 Tensorflow 和 GPU  
Hub 在 Docker 容器中運行  
每個使用者的筆記本（JupyterLab）都在獨立的 Docker 容器中啟動  
共享資料夾（例如使用者的家目錄和資料夾）被掛載到使用者的 Docker 容器，以保持資料的持久性  

## 軟體配置
Nvidia Cuda 驅動程式（目前為 nvidia-driver-535，由 Ubuntu "Additional Drivers" 應用程式安裝）  
docker（24.0.2）   
docker-compose（2.3.0）    
nvidia-docker2（2.13.0）    

## 初始步驟
參考https://cschranz.medium.com/set-up-your-own-gpu-based-jupyterlab-e0d45fcacf43  
在主機上創建一個新使用者（例如 jupyterhub），用於管理 JupyterHub 和存放 Hub 的配置文件    
將您的使用者加入 docker 群組，以便在不使用 sudo 的情況下執行 docker 命令   
通過運行 docker run --gpus all nvidia/cuda:11.6.2-cudnn8-runtime-ubuntu20.04 nvidia-smi 測試 nvidia-docker 中的 GPU 是否工作  
在主機上配置必要的共享資料夾掛載，例如 /home/jh_users/ 用於存放使用者的家目錄（我們將其與 UNIX 主機使用者分開），以及 /mnt/sdb2/ 用於共享資料（數據集、模型等）  

## 運行 JupyterHub
首次運行時，轉到根目錄並運行：  
docker-compose up -d (可選的 -d 參數將 Hub 作為服務運行)  
第一次運行此命令時，將構建 jupyterhub 映像並在 <host-ip>:8000 上運行服務器。默認登錄帳號為：jhadmin，密碼在 ./jupyterhub/Dockerfile 中設置。  


##警告：一旦您第一次運行容器，或者每次更新配置後，請立即將管理員密碼更改為安全密碼！要這樣做，運行 docker exec -it jupyterhub-container /bin/bash，然後運行 passwd jhadmin 選擇新密碼。

下次可以使用 docker-compose start 和 docker-compose stop 啟動/停止容器（例如在更新 jupyterhub_config.py 時）。

要停止並移除容器，請使用 docker-compose down。

## 添加新使用者
1.在主機上運行 docker exec -it jupyterhub-container /bin/bash  
2.運行 adduser <username>  
3.運行 passwd <username> 並設置密碼  
(option)在 Web 介面中以管理員身份登錄，在管理員菜單（頂部欄）中添加新使用者 <username>  


## 使用者環境
使用者可以使用 JupyterLab 的實例，設定以下資料結構：  

- ./work：啟用了持久性的工作目錄，它保存在主機機器上的 /home/jh_users/{username}/中  
- ./SHARED：共享的數據文件夾以只讀方式掛載，它保存在主機機器上的 /mnt/sda2/中  

