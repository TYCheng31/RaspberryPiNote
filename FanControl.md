# RaspberryPi5 FanControl


## 1.建立自動風扇控制腳本
```
sudo nano /usr/local/bin/fan_control.sh
```
## 2.貼上以下腳本
```
#!/bin/bash

# **設定風扇 PWM 控制路徑**
FAN_PWM_PATH="/sys/class/hwmon/hwmon3/pwm1"  

# **溫度閾值（單位：毫攝氏度）**
ON_TEMP=50000   # 50°C 開啟風扇
OFF_TEMP=40000  # 40°C 關閉風扇
MID_TEMP=60000  # 60°C 以上高速運行

# **風扇速度（0=關閉，255=最高速）**
FAN_OFF=0
FAN_LOW=200   # 50% 轉速
FAN_HIGH=255  # 100% 轉速

while true; do
    # **讀取 CPU 溫度**
    CPU_TEMP=$(cat /sys/class/thermal/thermal_zone0/temp)

    # **根據溫度設定風扇速度**
    if [ "$CPU_TEMP" -ge "$MID_TEMP" ]; then
        echo $FAN_HIGH | sudo tee $FAN_PWM_PATH
    elif [ "$CPU_TEMP" -ge "$ON_TEMP" ]; then
        echo $FAN_LOW | sudo tee $FAN_PWM_PATH
    elif [ "$CPU_TEMP" -le "$OFF_TEMP" ]; then
        echo $FAN_OFF | sudo tee $FAN_PWM_PATH
    fi

    sleep 30  # **每 10 秒檢查一次溫度**
done
```
## 3. 設定腳本可執行
```
sudo chmod +x /usr/local/bin/fan_control.sh
```
## 4. 測試腳本
```
sudo /usr/local/bin/fan_control.sh
```
## 5. 讓腳本開機自動運行
```
sudo nano /etc/systemd/system/fan_control.service
```
## 6. 貼上以下內容
```
[Unit]
Description=Raspberry Pi 5 Fan Control
After=multi-user.target

[Service]
Type=simple
ExecStart=/usr/local/bin/fan_control.sh
Restart=always
User=root

[Install]
WantedBy=multi-user.target
```
## 7. 啟用並啟動服務
```
sudo systemctl enable fan_control
sudo systemctl start fan_control
```
## 7-1.確認服務是否運行
```
sudo systemctl status fan_control
(如果看到 "active (running)"，就代表風扇已經可以自動運作了！)
```
