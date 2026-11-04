## Allow UDisks2 Mounting via Polkit (KDE Auto-Mount)

###1 1. Create a custom Polkit rule file:
```bash
sudo nano /etc/polkit-1/rules.d/10-udisks2.rules
```

### 2. Paste the following configuration:
```bash
polkit.addRule(function(action, subject) {
    if ((action.id == "org.freedesktop.udisks2.filesystem-mount-system" ||
         action.id == "org.freedesktop.udisks2.filesystem-mount") &&
        subject.isInGroup("sudo")) {
        return polkit.Result.YES;
    }
});
```
> (Note: If your distribution uses the wheel group instead of sudo for administrator privileges, replace "sudo" with "wheel").

### 3. Restart the Polkit service:
```bash
sudo systemctl restart polkit
```
> Verification: Log out and log back in, or unmount the partition in Dolphin and click it to mount. It should mount immediately without prompting for a password.
