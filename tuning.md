# Tuning Tools

## LACT
```bash
pikman update && pikman install lact

# Enable and Start the Background Daemon
sudo systemctl enable --now lactd
```
## CPU Frequency limit
```bash
pikman install linux-cpupower

sudo nano /etc/systemd/system/cpupower-cap.service
```
