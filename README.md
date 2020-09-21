# Nginx Server

  
                                Moduel  |  Data |  File
                                ________|_______|_________
                                Registry|       |  Stack
                                
                                
                                           ||                 Master Process
                                           
                                                               （實現平行處理）
                                
                             worker threads | slave process   slave process   slave process      全域：設定允許產生的工作處理程序的數量
                             
                                                 /||\               /||\             /||\         
                                                / || \             / || \           / || \       事件：connections 單一工作程序能處理的最大連接數設定  65535  
                                               /  || \            /  ||  \         /  ||  \    
                                               
                                            ｜｜｜｜｜｜｜ ｜｜｜｜｜｜  ｜｜｜｜｜｜ ｜｜｜｜｜｜ ｜｜   Http: 單連接請求上限數                                        




# 伺服器的設定 

 * Events, 允許多個網路連接
 
        multi_accept on | off;
 
       工作處理程序有能力同時街都到多個新到達的網路連接。(如下可避免競奪)

 * Events, 設定網路連接序列化
 
        accept_mutex on;
        
        雖然主處理程序能適切分配資源和指令予工作處理程序，然而網路的連接需求可能對多個工作處理程序產生競奪風險（同時多條工作處理程序被喚醒，雖然只有其中一條會實際執行任務），但多條被喚醒的主處理程序會造成系統的負擔，故網路連接變成序列化，防止多個工作處理程序對連接爭搶。
 
 * 選擇事件模型
 
       use method;
 
 * Global, 設定允許產生的工作處理程序的數量（實現平行處理）
 
        worker_process 3 | auto | 1;
 
        此功能能實現伺服器平行處理服務的關鍵所在， worker_process 的數量越多，能支援平行處理的數量也越多。
        但此數量上限值受限於軟體、作業系統資源、硬體裝置（運算器、磁碟機）。
        
        以上設定數量，為工作處理程序有三條，外加主處理程序一條。

 * Events, 單一工作處理程序的最大連接數設定
 
         worker_connections 65535;
 
         Client = worker_process * worker_connections / 2;
         
         其中 worker_connections 是指每條工作處理程序同時連接用戶端的最大數量，預設為 65535。
         建議設定為 65535，約莫一條工作處理程序可以做 65535 條連結到相應的用戶端，除以來回一趟，約莫服務 32467 個用戶端。
         此工作處理程序連結數量需要小於 open file resource limit (工作處理程序可以開啟的檔案控制代碼數量)，建議將此資源限制數量設定為 2390251。
         
 * Http, log 自訂記錄檔 
 
 * Global, 設定檔的引用
 
        寫後端會碰到，略
        
 * Http, 檔案引用
 
 * 設定錯誤檔儲存路徑
 
        寫後端會碰到，略
         
 * 網站預設首頁
 
        寫後端會碰到，略
 
 * 網站錯誤頁面
 
        寫後端會碰到，略

 * 設定請求的根目錄
 
 * location 的 URI
 
 * 設定虛擬主機
 
     * IP
     
     * DNS
     
 * Global, 設定伺服器使用群（組）

        user admin | group;
        
        只有被設定的使用者或是使用者群組才擁有許可權啟動伺服器的主處理程序
      
 * Http, MIME-Type
      
 * time_out
 
     * keepalive_timeot
     
     * Http, send_timeout
     
            設定伺服器回應用戶端的逾時間，這時間是指伺服器和用戶端建立連線後，某活動之後的時間，
            倘若此時間過後，用戶端無任何活動（伺服器等待用戶端回應超過此時間），伺服器則關閉連線 
            (L5 Session Layer 會議層含 Socket 通訊端)。
 
 * buffer_size
 
     * getconf PAGESIZE;
     
            設定伺服器允許用端端請求標頁首的緩衝區大小，預設為 1 KB（根據系統分頁大小可調整不同）
     
     * client_header_buffer_size;
     
            用戶端可能因為 cookie 中寫入較大的值，引發用戶端請求標頁首大於預設的 1 KB ，
            為了改善伺服器支援的功能，建議預設值調整為 4 KB。
      
 * Http, 單連接請求數上限
      
 * 造訪權限設定
 
     * IP
     
     * 密碼
      
 * CPU
      
 * Core
      
 * sigpending 事件訊號佇列長度的上限
 
        每一個工作處理程序有自己事件訊號佇列用於暫存用戶端請求發生訊號，倘若超過長度上限，伺服器會轉用 poll 處理尚未處理的用戶端請求。為了確保用戶端平行處理請求數量和伺服器執行環境處理能力，建議預設為 1024。
        
 * 設定網路監聽
 
 * Http, 設定允許的方式傳輸檔案

# 設定檔的結構

nginx.conf 文件中
為洋蔥式層級，分為幾層大括號。

      global { 
      
                  全區塊能預設設定檔案從開始到 events 區塊之間的一部分內容，
                  這些指令影響伺服器全域。

                  範疇包含：

                  （1)伺服器使用者群組
                   (2)允許產生的工作處理程序數量
                   (3)處理程序PID儲存路徑
                   (4)log 儲存路徑
                   (5)設定檔引用
                   

              events {
              
                  
                      event 區塊指令會影響伺服器與使用者的網路連接。
                      設定網路連接序列化、是否允許多個網路連接、單一工作處理程序的最大連接數便是處於此層級



                    http {
                    
                           設定指令包含：
                           (1) MIMI-Type 定義
                           (2) log 自訂記錄檔
                           (3) 設定允許的方式傳輸檔案
                           (4) 單連接請求上線數
                           (5) 連接逾時間

                         (虛擬主機)server1 {
                         
                         
                                   虛擬主機伺服器又稱為虛擬伺服器、主機空間，這裡的虛擬伺服器
                                   是利用實際硬體伺服器衍生出來的主機或空間。
                                   
                                   虛擬技術主要應用於 Http、Ftp、Enail 這些常見的服務，
                                   將一台伺服器的某服務內容邏輯劃分出多個服務單位，
                                   對外像是多個伺服器，也充分利用的一台硬體資源。
                                   
                                   而從使用者角度來看，一台虛擬伺服器和一台硬體伺服器，
                                   其實是一樣的。
                                   
                                   虛擬主機技術能使同一台伺服器執行（多台虛擬伺服器）一組主處理程序
                                   就可以執行多個網站。
                             
                         
                               location {
                             
                                     
                                      虛擬伺服器中可以擁有很多的 location 區塊，其實 location 是虛擬伺服器的指令。
                                      區塊能展現伺服器的靈活度：
                                      
                                      (1)此處能讓虛擬伺服器在收到基礎的請求字串
                                      如：server_name/uri-string，對除了虛擬主機名稱（亦可為 IP alias)外，
                                      對其 uri-string 字串能進行比對，對特定請求進行處理。
                                      
                                      (2)位址定向                     
                                      (3)資料快取                      
                                      (4)回應控制
                                      
                               
                               }
                               
                               
                               location {
                               
                                                              
                               }
                               
                               
                               location {
                                                   
                               
                               }
                               
                               
                               location {
                                                                   
                               
                               }
                               

                         }
                         
                      ------------------------------------------
                         
                          (虛擬主機)server2 {
                         
                              每台虛擬伺服器的作用域都僅僅針對個別虛擬伺服器
                    
                             
                               location {
                               
                               
                               
                               
                               }
                               
                              

                          }
                          
                        ------------------------------------------
                          
                          
                       (虛擬主機)server3 {
                            
                                
                                每台虛擬伺服器的作用域都僅僅針對個別虛擬伺服器
                         
                         
                               location {
                               
                               
                               
                               
                              }

                      }
                      
                     
                     ------------------------------------------


              } # Event

      } # Global

