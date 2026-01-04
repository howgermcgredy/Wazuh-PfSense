# Wazuh-PfSense
# Wazuh SIEM 與 pfSense 防火牆監控整合實作

## 專案簡介 (Introduction)
本專案實作了一套基於 **Wazuh (SIEM)** 與 **pfSense (Firewall)** 的資安監控環境。目標是透過 Agentless (無代理) 架構，將邊界防火牆的日誌 (Syslog) 即時傳送至 Wazuh Manager 進行解析、偵測與視覺化分析。

## 系統架構 (Architecture)
* **Firewall**: pfSense Community Edition (VirtualBox)
  - WAN: NAT Network
  - LAN: 192.168.1.1 (負責內網閘道)
* **SIEM Server**: Ubuntu 22.04 LTS + Wazuh Manager
  - IP: 192.168.1.100
  - Role: Log Receiver & Analyzer

## 實作細節 (Implementation)

1. 資料流建立 (Log Transport)
在 pfSense 設定 Remote Syslog，透過 UDP/514 端口將日誌轉發至 Wazuh Server。
驗證方式：使用 `tcpdump` 確認封包到達。
![Tcpdump Proof](assets/02_tcpdump_verify.png)

2. 規則配置 (Custom Rules)
為了正確解析 pfSense 事件，於 Wazuh 編寫了自定義 XML 規則 (`local_rules.xml`)：
```xml
<group name="local,syslog,">
  <rule id="100001" level="5">
    <description>pfSense Firewall Event (Lab Test)</description>
    <match>pfSense</match>
  </rule>
</group>
```

3. 偵測成果 (Detection Results)
成功在 Wazuh Dashboard 偵測到防火牆的登入與操作事件。

## 未來展望：自動化回應 (Active Response)

# Wazuh-PfSense
若要達成「自動阻擋」，計畫在 Wazuh 配置 Active Response 腳本，透過 SSH Key 連線至 pfSense 執行 easyrule block <Src_IP>，實現 IP 自動封鎖。
1. 設定檔提取
把虛擬機裡面的檔案複製出來，變成真正的文字檔。
* **`local_rules.xml`**：
    在 Ubuntu 終端機執行 `cat /var/ossec/etc/rules/local_rules.xml`，把顯示出來的內容複製，在電腦新增一個同名文字檔貼上。
* **`ossec.conf`**：
    同樣方式，複製 `/var/ossec/etc/ossec.conf` 的內容。
