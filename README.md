# binderfs-pve

early development release of binderfs for redroid on proxmox
full project to come if this works for other. 

## built for kernel 6.17.13-2-pve
may need to be rebuilt for other kernel releases

### 1. Build binderfs using `binderfs-mkdevs`
```
modprobe binder_linux
mkdir -p /dev/binderfs
mount -t binder binder /dev/binderfs
/usr/local/sbin/binderfs-mkdevs
```

You should now see:
```
ls /dev/binderfs
binder  hwbinder  vndbinder  binder-control  features
```

### 2. Create a binder group
```bash
groupadd binder
getent group binder
```

Example:
```
binder:x:637:
```

### 3. Apply that GID to binderfs devices
Use a udev rule so it persists:

`/etc/udev/rules.d/99-binderfs.rules`
```
KERNEL=="binder", GROUP="binder", MODE="0660"
KERNEL=="hwbinder", GROUP="binder", MODE="0660"
KERNEL=="vndbinder", GROUP="binder", MODE="0660"
KERNEL=="binder-control", GROUP="binder", MODE="0660"
```

Reload:
```bash
udevadm control --reload
udevadm trigger
```

### 4. Map that GID into your LXC
In your container config:
```
dev2: /dev/binderfs/binder,gid=637
dev3: /dev/binderfs/hwbinder,gid=637
dev4: /dev/binderfs/vndbinder,gid=637
```

### 5. Recreate at startup
`/etc/systemd/system/binderfs.service`
```
[Unit]
Description=BinderFS mount and device creation
After=local-fs.target
DefaultDependencies=no

[Service]
Type=oneshot
ExecStart=/sbin/modprobe binder_linux
ExecStart=/bin/mkdir -p /dev/binderfs
ExecStart=/bin/mount -t binder binder /dev/binderfs
ExecStart=/usr/local/sbin/binderfs-mkdevs
RemainAfterExit=yes

[Install]
WantedBy=multi-user.target
```

Start it up

```
systemctl daemon-reload
systemctl enable --now binderfs.service
```
