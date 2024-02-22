# Dashboard OpenSearch + Notify Telegram
## Yêu cầu
1. Setup stack Prometheus + Grafana trên server LAB Monitor
2. Setup OpenSearch Exporter + Node Exporter trên hệ thống LAB siem-lab.usec.vn (soc_lab_wazuh_ao1)
3. Setup AlertManager notify Telegram: Hardware + healthcheck Opensearch
## Mô hình
![Mo hinh](/Picture/Mo%20hinh.png)
## Cách cài đặt tích hợp
### 1. Setup Prometheus + Grafana + Node Exporter
- Download, unzip và cài đặt:

    Prometheus [here](https://github.com/prometheus/prometheus/releases/download/v2.43.0/prometheus-2.43.0.linux-amd64.tar.gz)
    
    Grafana [here](https://dl.grafana.com/enterprise/release/grafana-enterprise_9.4.7_amd64.deb)

    Node-Exporter [here](https://gitlab.vsec.vn/lamtruong/dashboard-opensearch-alertmanager-telegram/-/blob/main/Node-Exporter/docker-compose.yml)

- Cấu hình và khởi động dịch vụ:

    [Prometheus as a service](https://gitlab.vsec.vn/lamtruong/dashboard-opensearch-alertmanager-telegram/-/blob/main/Prometheus/prometheus.service) **(Lưu ý tạo file log và phân quyền trước khi start service)**. Di chuyển folder prometheus vừa unzip bước trên vào **/usr/local/bin**. Chuyển file **prometheus.yml** ra folder mới để tiện config [/etc/prometheus/prometheus.yml](https://gitlab.vsec.vn/lamtruong/dashboard-opensearch-alertmanager-telegram/-/blob/main/Prometheus/prometheus.yml). Đồng thời thêm các [alert rules](https://gitlab.vsec.vn/lamtruong/dashboard-opensearch-alertmanager-telegram/-/tree/main/Prometheus/alert) vào cùng folder đó.

    Sau khi khởi chạy node-exporter và prometheus thì vào web Grafana add data source prometheus. Sau đó import ID dashboard node-exporter 11074 để check hardware server.

### 2. Setup OpenSearch Exporter (soc_lab_wazuh_ao1)
- Tạo folder **exporter** rồi tạo file [docker-compose.yml](https://gitlab.vsec.vn/lamtruong/dashboard-opensearch-alertmanager-telegram/-/blob/main/Opensearch-Exporter/docker-compose.yml) để chạy exporter gồm 2 service **opensearch-exporter** và **node-exporter**.

### 3. Setup AlertManager notify Telegram
- **Telegram Bot**

    Tạo bot mới trên Telegram bằng BotFather (tự search các tạo). Add bot vào group sau đó chat lần đầu để lấy **chat id** check trên https://api.telegram.org/bot(token)/getUpdates.

    Tạo folder cho service **telegram_bot** gồm 2 phần:

    - File [docker-compose.yml](https://gitlab.vsec.vn/lamtruong/dashboard-opensearch-alertmanager-telegram/-/blob/main/Telegram%20Bot/docker-compose.yml)
    
    - Folder con tên [telegrambot](https://gitlab.vsec.vn/lamtruong/dashboard-opensearch-alertmanager-telegram/-/tree/main/Telegram%20Bot/telegrambot) chứa 2 file: config + template.

- **AlertManager**

    Download và unzip: AlertManager [here](https://github.com/prometheus/alertmanager/releases/download/v0.25.0/alertmanager-0.25.0.linux-amd64.tar.gz)

    [AlertManager as a service](https://gitlab.vsec.vn/lamtruong/dashboard-opensearch-alertmanager-telegram/-/blob/main/AlertManager/alertmanager.service) **(Lưu ý tạo file log và phân quyền trước khi start service)**. Di chuyển folder **alertmanager** vừa unzip bước trên vào **/usr/local/bin**. Chuyển file **alertmanager.yml** ra folder mới để tiện config [/etc/alertmanager/alertmanager.yml](https://gitlab.vsec.vn/lamtruong/dashboard-opensearch-alertmanager-telegram/-/blob/main/AlertManager/alertmanager.yml) (lưu ý trong **url webhook_configs** sửa **chat id** đã lấy được từ bước tạo telegram bot).

- Sau khi khởi động xong 2 dịch vụ trên chúng ta sẽ lên web Grafana rồi import [template](https://gitlab.vsec.vn/lamtruong/dashboard-opensearch-alertmanager-telegram/-/blob/main/Dashboard%20Template/Opensearch-overview.json) để hiển thị **Dashboard OpenSearch**

## Kết quả
- **Dashboard Opensearch**

![Dashboard Opensearch](https://gitlab.vsec.vn/lamtruong/dashboard-opensearch-alertmanager-telegram/-/raw/main/Picture/Opensearch%20Dashboard.png)

- **Nofify Telegram**

![Bot Telegram Notify](https://gitlab.vsec.vn/lamtruong/dashboard-opensearch-alertmanager-telegram/-/raw/main/Picture/Notify%20Telegram%20Bot.png)

- **Các thông số cảnh báo về Telegram**:
    
    **Node-exporter**:
    
    - RAM
    - CPU
    - Disk Space (Write, Read)
    - Network (IN, OUT)

    **OpenSearch-exporter**:
    
    - Missing node
    - Missing data node
    - Heap Too High
    - Cluster YELLOW, RED
    - Unassigned Shards
    - Pending Tasks (*)
    - JVM Heap Use High
    - Process CPU High

## Cải thiện và nâng cấp
- Khắc phục định dạng alert bot trả kết quả:

    Một số chỗ còn chưa rõ ràng do template rules alert chưa chuẩn
    
    Tối ưu lại description và summary

- Tích hợp thêm Blackbox exporter để cải thiện các thông báo trùng lặp trong rules Opensearch exporter