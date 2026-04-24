````markdown
# ZFS Lab: Compression, Pool Configuration, and Snapshots

## 📌 Project Title
ZFS Storage Pools: Compression Efficiency, Pool Configuration, and Snapshot Recovery

---

## 📖 Introduction
This project demonstrates working with the ZFS filesystem:
- Comparing compression algorithms
- Creating and analyzing ZFS pools
- Importing an existing pool
- Working with snapshots and data recovery

---

## 📑 Table of Contents
- [Installation](#installation)
- [Disk Layout](#disk-layout)
- [Pool Creation](#pool-creation)
- [Compression Comparison](#compression-comparison)
- [Best Compression Algorithm](#best-compression-algorithm)
- [Pool Import and Configuration](#pool-import-and-configuration)
- [Snapshots and Recovery](#snapshots-and-recovery)
- [Results](#results)
- [Troubleshooting](#troubleshooting)
- [Dependencies](#dependencies)
- [License](#license)

---

## ⚙️ Installation

Install ZFS utilities:

```bash
sudo apt update
sudo apt install zfsutils-linux
````

---

## 💽 Disk Layout

```bash
lsblk
```

Available disks:

* System disk: `sda`
* Additional disks: `sdb`–`sdi` (512MB each)

---

## 🏗️ Pool Creation

Create 4 ZFS pools in RAID1 (mirror mode):

```bash
zpool create otus1 mirror /dev/sdb /dev/sdc
zpool create otus2 mirror /dev/sdd /dev/sde
zpool create otus3 mirror /dev/sdf /dev/sdg
zpool create otus4 mirror /dev/sdh /dev/sdi
```

Check pools:

```bash
zpool list
```

---

## 🗜️ Compression Comparison

Set different compression algorithms:

```bash
zfs set compression=lzjb otus1
zfs set compression=lz4 otus2
zfs set compression=gzip-9 otus3
zfs set compression=zle otus4
```

Verify:

```bash
zfs get compression
```

---

## 📥 Test File Download

Download the same file into each pool:

```bash
for i in {1..4}; do
  wget -P /otus$i https://gutenberg.org/cache/epub/2600/pg2600.converter.log
done
```

---

## 📊 Compression Results

```bash
zfs list
zfs get compressratio
```

| Pool  | Algorithm | Used Space | Compression Ratio |
| ----- | --------- | ---------- | ----------------- |
| otus1 | lzjb      | 21.8M      | 1.82x             |
| otus2 | lz4       | 17.7M      | 2.23x             |
| otus3 | gzip-9    | 10.9M      | **3.66x**         |
| otus4 | zle       | 39.5M      | 1.00x             |

---

## 🏆 Best Compression Algorithm

**gzip-9 provides the highest compression ratio (~3.66x)**

⚠️ Note:

* `gzip-9` → best compression, slowest
* `lz4` → best performance (recommended in production)

---

## 📦 Pool Import and Configuration

### Download archive

```bash
wget -O archive.tar.gz --no-check-certificate '<URL>'
tar -xzvf archive.tar.gz
```

---

### Import pool

```bash
zpool import -d zpoolexport/
zpool import -d zpoolexport/ otus
```

---

### Pool Status

```bash
zpool status
```

Result:

* Pool name: `otus`
* Type: **mirror (RAID1)**
* State: ONLINE

---

## 🔍 Pool Configuration

### Storage Size

```bash
zpool list otus
```

---

### Pool Type

```bash
zpool status otus
```

Result:

```
mirror-0 → mirror (RAID1)
```

---

### Record Size

```bash
zfs get recordsize otus
```

Typical value:

```
128K
```

---

### Compression

```bash
zfs get compression otus
```

---

### Checksum

```bash
zfs get checksum otus
```

Typical value:

```
sha256
```

---

## 📸 Snapshots and Recovery

### List snapshots

```bash
zfs list -t snapshot
```

---

### Copy file from hidden directory

Search for file:

```bash
find /otus -name "*secret*"
```

---

### Restore using ZFS receive

```bash
zfs receive otus < snapshot_file
```

---

### Rollback to snapshot

```bash
zfs rollback otus@snapshot_name
```

---

### Extract secret message

```bash
cat secret_message
```

If binary:

```bash
strings secret_message | grep -i secret
```

---

## ✅ Results

* Best compression: **gzip-9**
* Pool type: **mirror**
* Record size: **128K**
* Compression: depends on pool setting
* Checksum: **sha256**

---

## 🛠️ Troubleshooting

* If pool import fails:

```bash
zpool import -f -d <dir>
```

* If features are missing:

```bash
zpool upgrade otus
```

⚠️ Warning: upgrade is irreversible

---

## 📦 Dependencies

* Ubuntu / Linux
* ZFS (`zfsutils-linux`)
* wget
* tar

---

## 📄 License

This project is for educational purposes.

```
```

