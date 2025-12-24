# LERNING TRAEFIK


## DOWNLOAD

```
https://github.com/traefik/traefik/releases
```

## LINUX

```
https://github.com/traefik/traefik/releases/download/v3.6.5/traefik_v3.6.5_linux_amd64.tar.gz
```

## WINDOWS

```
https://github.com/traefik/traefik/releases/download/v3.6.5/traefik_v3.6.5_windows_amd64.zip
```

## START WEB

```
node server.js 5001
node server.js 5002
node server.js 5003
node server.js 5005
```

## START COMMAND 

```bat
.\traefik.exe --configFile=traefik.yml
```

====

Here is the step-by-step guide to installing **Traefik v3.6.5** from your specific tarball on **Ubuntu 24.04**.

### **Step 1: Extract and Install the Binary**

First, ensure you are in the directory where your file `traefik_v3.6.5_linux_amd64.tar.gz` is located.

1. **Extract the archive:**
```bash
tar -zxvf traefik_v3.6.5_linux_amd64.tar.gz

```


*(This usually produces a single binary file named `traefik`)*
2. **Move the binary to the system path:**
Standard practice is to place manual binaries in `/usr/local/bin`.
```bash
sudo mv traefik /usr/local/bin/

```


3. **Give it executable permissions:**
```bash
sudo chmod +x /usr/local/bin/traefik

```



---

### **Step 2: Create a Dedicated User**

For security, you should not run Traefik as `root`. Create a system user specifically for Traefik.

```bash
sudo useradd -r -s /bin/false -U -M traefik

```

* `-r`: System account.
* `-s /bin/false`: No login shell (security).
* `-U`: Create a user group with the same name.
* `-M`: Do not create a home directory.

---

### **Step 3: Allow Port Binding (80/443)**

By default, Linux prevents non-root users from binding to ports below 1024. You can allow Traefik to bind to port 80 (HTTP) and 443 (HTTPS) without being root by giving it specific capabilities.

```bash
sudo setcap 'cap_net_bind_service=+ep' /usr/local/bin/traefik

```

---

### **Step 4: Create Configuration Directory**

Traefik needs a place to store its configuration.

1. **Create the directory:**
```bash
sudo mkdir /etc/traefik
sudo chown -R root:root /etc/traefik

```


2. **Create a basic configuration file:**
Create a file named `traefik.yml` (YAML format is standard for v3).
```bash
sudo nano /etc/traefik/traefik.yml

```


3. **Paste this basic configuration:**
```yaml
# /etc/traefik/traefik.yml

api:
  dashboard: true
  insecure: true # WARNING: Don't use 'insecure: true' in production exposed to the internet!

entryPoints:
  web:
    address: ":80"
  websecure:
    address: ":443"

providers:
  file:
    directory: "/etc/traefik/dynamic"
    watch: true

```


*(Save with `Ctrl+O`, `Enter`, then `Ctrl+X`)*
4. **Create the dynamic config folder:**
```bash
sudo mkdir /etc/traefik/dynamic

```



---

### **Step 5: Create a Systemd Service**

To ensure Traefik starts automatically on boot, create a systemd service file.

1. **Create the service file:**
```bash
sudo nano /etc/systemd/system/traefik.service

```


2. **Paste the following content:**
```ini
[Unit]
Description=Traefik v3.6.5
Documentation=https://doc.traefik.io/traefik/
After=network-online.target
Wants=network-online.target systemd-networkd-wait-online.service

[Service]
# Run as the dedicated user we created
User=traefik
Group=traefik

# Path to the binary and config file
ExecStart=/usr/local/bin/traefik --configfile=/etc/traefik/traefik.yml

# Restart automatically if it crashes
Restart=always

# Security capabilities to allow binding to port 80/443
AmbientCapabilities=CAP_NET_BIND_SERVICE
CapabilityBoundingSet=CAP_NET_BIND_SERVICE

# Hardening (Security Best Practices)
ProtectSystem=full
ProtectHome=true
PrivateTmp=true
NoNewPrivileges=true

[Install]
WantedBy=multi-user.target

```



---

### **Step 6: Start and Enable Traefik**

1. **Reload systemd to recognize the new service:**
```bash
sudo systemctl daemon-reload

```


2. **Enable Traefik to start on boot:**
```bash
sudo systemctl enable traefik

```


3. **Start Traefik now:**
```bash
sudo systemctl start traefik

```


4. **Check the status:**
```bash
sudo systemctl status traefik

```


*You should see a green "active (running)" status.*

### **Verification**

If you used the configuration above (with `insecure: true`), you can access the dashboard by opening your browser and navigating to:
`http://<your-server-ip>:8080`

**Next Step:** Would you like a guide on how to configure Traefik to serve a specific application or how to set up Let's Encrypt SSL certificates automatically?

===

นี่คือ Shell Script สำหรับติดตั้ง **Traefik v3.6.5** บน Ubuntu 24.04 แบบอัตโนมัติครับ

สคริปต์นี้จะจัดการตั้งแต่แตกไฟล์, สร้าง User, ตั้งค่า Permission, สร้างไฟล์ Config, และตั้งค่า Systemd Service ให้ครบจบในขั้นตอนเดียว

### วิธีใช้งาน

1. สร้างไฟล์ชื่อ `install_traefik.sh`
2. ก๊อปปี้โค้ดด้านล่างลงไป
3. วางไฟล์ `traefik_v3.6.5_linux_amd64.tar.gz` ไว้ในโฟลเดอร์เดียวกันกับสคริปต์
4. รันคำสั่ง: `sudo bash install_traefik.sh`

---

### โค้ด `install_traefik.sh`

```bash
#!/bin/bash

# กำหนดตัวแปร
TRAEFIK_VERSION="v3.6.5"
FILENAME="traefik_${TRAEFIK_VERSION}_linux_amd64.tar.gz"
INSTALL_DIR="/usr/local/bin"
CONFIG_DIR="/etc/traefik"
SERVICE_FILE="/etc/systemd/system/traefik.service"

# 1. ตรวจสอบสิทธิ์ Root
if [ "$EUID" -ne 0 ]; then
  echo "❌ กรุณารันด้วย sudo หรือ root"
  exit 1
fi

# 2. ตรวจสอบว่ามีไฟล์ติดตั้งหรือไม่
if [ ! -f "$FILENAME" ]; then
    echo "❌ ไม่พบไฟล์ $FILENAME ในโฟลเดอร์ปัจจุบัน"
    exit 1
fi

echo "🚀 เริ่มการติดตั้ง Traefik $TRAEFIK_VERSION..."

# 3. แตกไฟล์และย้ายไปที่ /usr/local/bin
echo "📦 กำลังแตกไฟล์..."
tar -zxvf "$FILENAME" traefik
mv traefik "$INSTALL_DIR/"
chmod +x "$INSTALL_DIR/traefik"

# 4. สร้าง User สำหรับรัน Traefik (เพื่อความปลอดภัย)
if ! id "traefik" &>/dev/null; then
    echo "👤 สร้าง User: traefik"
    useradd -r -s /bin/false -U -M traefik
fi

# 5. อนุญาตให้ bind port 80/443 โดยไม่ต้องเป็น root
echo "🔒 ตั้งค่า Permission (setcap)..."
# ติดตั้ง libcap2-bin หากยังไม่มี (จำเป็นสำหรับ setcap)
apt-get update -qq && apt-get install -y libcap2-bin
setcap 'cap_net_bind_service=+ep' "$INSTALL_DIR/traefik"

# 6. สร้างโฟลเดอร์ Config และไฟล์ traefik.yml พื้นฐาน
echo "⚙️ สร้างไฟล์ Config..."
mkdir -p "$CONFIG_DIR/dynamic"

# สร้างไฟล์ traefik.yml (Static Config)
cat <<EOF > "$CONFIG_DIR/traefik.yml"
api:
  dashboard: true
  insecure: true # ⚠️ คำเตือน: สำหรับ Production ควรตั้งเป็น false และใช้ Middleware ป้องกัน

entryPoints:
  web:
    address: ":80"
    http:
      redirections:
        entryPoint:
          to: websecure
          scheme: https
  websecure:
    address: ":443"

providers:
  file:
    directory: "$CONFIG_DIR/dynamic"
    watch: true
EOF

# เปลี่ยนเจ้าของโฟลเดอร์เป็น root (เพื่อความปลอดภัยไฟล์ config ไม่ควรแก้โดย user ทั่วไป)
chown -R root:root "$CONFIG_DIR"

# 7. สร้าง Systemd Service File
echo "📝 สร้าง Systemd Service..."
cat <<EOF > "$SERVICE_FILE"
[Unit]
Description=Traefik $TRAEFIK_VERSION
Documentation=https://doc.traefik.io/traefik/
After=network-online.target
Wants=network-online.target systemd-networkd-wait-online.service

[Service]
# Run as dedicated user
User=traefik
Group=traefik

# Path to binary and config
ExecStart=$INSTALL_DIR/traefik --configfile=$CONFIG_DIR/traefik.yml

# Restart policy
Restart=always
RestartSec=10

# Security Capabilities (Bind ports < 1024)
AmbientCapabilities=CAP_NET_BIND_SERVICE
CapabilityBoundingSet=CAP_NET_BIND_SERVICE

# Hardening
ProtectSystem=full
ProtectHome=true
PrivateTmp=true
NoNewPrivileges=true

[Install]
WantedBy=multi-user.target
EOF

# 8. เริ่มการทำงาน
echo "🔥 กำลังเริ่ม Service..."
systemctl daemon-reload
systemctl enable traefik
systemctl start traefik

# 9. ตรวจสอบสถานะ
if systemctl is-active --quiet traefik; then
    echo "✅ ติดตั้ง Traefik สำเร็จ! สถานะ: Active"
    echo "🌐 Dashboard (ถ้าเปิด insecure): http://<YOUR-IP>:8080"
else
    echo "❌ เกิดข้อผิดพลาดในการเริ่ม Service กรุณาเช็ค: systemctl status traefik"
fi

```

**ขั้นตอนถัดไป:** หากสคริปต์ทำงานสำเร็จ คุณต้องการให้ผมแนะนำวิธีเขียนไฟล์ Config ในโฟลเดอร์ `dynamic` เพื่อเราท์ (Route) โดเมนไปยัง Docker Container หรือไม่ครับ?

