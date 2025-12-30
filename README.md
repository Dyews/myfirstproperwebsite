# Raspberry Pi WebRTC Camera Streaming (MediaMTX + rpicam)

This guide describes how to set up a **low-latency, free WebRTC live camera stream**
on a Raspberry Pi using **MediaMTX**, **rpicam**, and **ffmpeg**.

The stream can be viewed directly in a browser and later exposed securely via
Cloudflare Tunnel.

---

## Hardware & OS

- **Device:** Raspberry Pi 3 A+
- **Camera:** OV5647
- **OS:** Raspberry Pi OS (64-bit, aarch64)
- **Network:** Local Wi-Fi (internet exposure optional)

---

## Architecture Overview

Camera (OV5647)
↓
rpicam-vid (H.264 hardware encoding)
↓
ffmpeg (RTSP publisher)
↓
MediaMTX (RTSP + WebRTC server)
↓
Browser (WebRTC playback)

yaml
Copy code

---

## Install Required Packages

```bash
sudo apt update
sudo apt install -y rpicam-apps ffmpeg
Installed tools:

rpicam-vid – Camera capture with hardware H.264 encoding

ffmpeg – Wraps H.264 stream into RTSP

libcamera – Underlying camera stack

Install MediaMTX (ARM64)
bash
Copy code
cd ~
wget https://github.com/bluenviron/mediamtx/releases/latest/download/mediamtx_linux_arm64.tar.gz
tar xvf mediamtx_linux_arm64.tar.gz
Install binary
bash
Copy code
sudo mkdir -p /opt/mediamtx
sudo mv mediamtx /opt/mediamtx/
sudo chmod +x /opt/mediamtx/mediamtx
MediaMTX Configuration
Create config directory
bash
Copy code
sudo mkdir -p /etc/mediamtx
sudo nano /etc/mediamtx/mediamtx.yml
/etc/mediamtx/mediamtx.yml
yaml
Copy code
webrtc: yes
webrtcAddress: :8889
api: yes
logLevel: info

paths:
  cam:
    source: publisher
Create Dedicated System User (Recommended)
bash
Copy code
sudo useradd -r -s /usr/sbin/nologin mediamtx
sudo chown -R mediamtx:mediamtx /opt/mediamtx /etc/mediamtx
This limits privileges and improves security.

MediaMTX systemd Service
Create service file
bash
Copy code
sudo nano /etc/systemd/system/mediamtx.service
ini
Copy code
[Unit]
Description=MediaMTX Streaming Server
After=network.target

[Service]
User=mediamtx
Group=mediamtx
ExecStart=/opt/mediamtx/mediamtx /etc/mediamtx/mediamtx.yml
Restart=always
RestartSec=2
LimitNOFILE=1048576

[Install]
WantedBy=multi-user.target
Enable & start
bash
Copy code
sudo systemctl daemon-reexec
sudo systemctl daemon-reload
sudo systemctl enable mediamtx
sudo systemctl start mediamtx
Verify
bash
Copy code
sudo systemctl status mediamtx
ss -ltnp | grep -E '(:8554|:8889)'
Expected listeners:

8554 – RTSP

8889 – WebRTC

Camera → RTSP Publisher Pipeline
Run manually (can later be converted into a service):

bash
Copy code
rpicam-vid \
  --width 640 --height 480 --framerate 15 \
  --codec h264 --inline --nopreview \
  -t 0 -o - \
| ffmpeg -re -f h264 -i - \
  -c copy -f rtsp rtsp://127.0.0.1:8554/cam
Local Testing
From another device on the same Wi-Fi, open:

arduino
Copy code
http://<PI_IP>:8889/cam
Example:

arduino
Copy code
http://192.168.1.226:8889/cam
You should see a low-latency live WebRTC stream.

Logs & Debugging
MediaMTX logs
bash
Copy code
journalctl -u mediamtx -f
Check open ports
bash
Copy code
ss -ltnp | grep -E '(:8554|:8889)'
Notes & Design Decisions
WebRTC provides low latency and native browser playback

H.264 hardware encoding keeps CPU usage low on Raspberry Pi 3

MediaMTX provides RTSP + WebRTC in a single lightweight binary

No paid services required

Suitable for personal or small-scale deployments

Next Steps (Optional)
Run the camera pipeline as a systemd service

Expose the stream via Cloudflare Tunnel

Embed the stream on a website

Secure access using Cloudflare Access

Add recording or snapshot functionality

yaml
Copy code

---

If you want, next I can:
- Add **Cloudflare Tunnel** steps directly into this README
- Add **ASCII or Mermaid diagrams**
- Add a **one-command installer script**
- Split it into **Setup / Troubleshooting / FAQ**

Just tell me how polished you want it 😄






