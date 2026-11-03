# Tuning Tools

## LACT
```bash
pikman update && pikman install lact

# Enable and Start the Background Daemon
sudo systemctl enable --now lactd
```
## CPU Frequency limit
```bash
# Install cpupower
pikman install linux-cpupower

# Create the Systemd Service File
sudo nano /etc/systemd/system/cpupower-cap.service


# Reload Systemd, Enable, and Start the Service
sudo systemctl daemon-reload
sudo systemctl enable --now cpupower-cap.service

# Verify the Hardware Frequency Limit
cpupower frequency-info
cat /sys/devices/system/cpu/cpu*/cpufreq/scaling_max_freq
```
