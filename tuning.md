# Tuning Tools

## LACT
```bash
pikman update && pikman install lact

# Enable and Start the Background Daemon
sudo systemctl enable --now lactd
```
## Frequency limit
```bash
pikman install linux-cpupower

nano /etc/systemd/system/cpupower-cap.service
```
