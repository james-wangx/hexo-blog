---
title: Install Oracle Database 19c on SLES 15 SP7
description: Install Oracle Database 19c on SUSE Linux Enterprise Server 15 SP7, create a CDB and PDB, configure Oracle Net, and enable systemd autostart.
date: 2026-11-08 21:04:45
tags:
  - Oracle
  - SLES
  - Database
  - Installation
categories:
  - Uncategorized
---

Oracle Database 19c is a common backend for Teamcenter. This guide explains how to perform a silent installation on SUSE Linux Enterprise Server (SLES) 15 SP7, create a container database (CDB) and a pluggable database (PDB), configure the listener, and enable automatic startup with systemd.

The example environment has the following layout:

| Item | Value |
| --- | --- |
| Operating system | SLES 15 SP7 x86-64 |
| Oracle release media | Oracle Database 19.3 base release |
| `ORACLE_BASE` | `/data/plm/oracle` |
| `ORACLE_HOME` | `/data/plm/oracle/product/19.0.0/dbhome_1` |
| Central Inventory | `/data/plm/oracle/inventory` |
| CDB name / SID | `tcdb` / `tcdb` |
| PDB name | `tcpdb` |
| Listener port | `1521` |
| Database character set | `AL32UTF8` |
| Database edition | Enterprise Edition |

## 1. Review the support boundary

Oracle Database 19c base media contains release 19.3. SLES 15 SP7 is newer than that base release and is not certified with an unpatched 19.3 database. The installation usually works, but Oracle and SUSE may not provide support for that exact combination.

For a production system, apply a certified 19c release update (RU), such as 19.27 or later, and verify the current Oracle certification matrix before deployment. Use this procedure for development or test systems when only the 19.3 base media is available.

## 2. Verify the operating system and resources

Confirm the operating system, architecture, hostname, memory, swap, and disk space:

```bash
cat /etc/os-release
uname -m
hostname
free -h
swapon --show
df -hT / /tmp /data
```

Use a hostname no longer than 31 characters. The database server should also have a stable IP address and a resolvable hostname.

The example system uses 16 GB of memory, 8 GB of swap, and local XFS storage. Do not put Oracle data files on NFS or another shared file system.

## 3. Mount the SLES Full ISO and add local repositories

Oracle requires libraries from several SLES modules. A Full ISO is the simplest source for an offline installation.

```bash
mkdir -p /mnt/sles15sp7
mount -o ro /dev/sr0 /mnt/sles15sp7

for repo_dir in $(find /mnt/sles15sp7 -maxdepth 3 \
  -type d -name repodata -printf '%h\n' | sort); do
  zypper --non-interactive ar --refresh \
    "dir:${repo_dir}" "local-$(basename "${repo_dir}")"
done

zypper --gpg-auto-import-keys refresh
zypper repos --details
```

The ISO contains the Basesystem, Server Applications, Development Tools, Desktop Applications, and Legacy modules. The Legacy module is required because Oracle 19c checks for `libcap1`.

## 4. Install the Oracle prerequisites

Install the runtime, linker, X11 client, and 32-bit compatibility packages:

```bash
zypper --non-interactive install \
  bc binutils make gcc gcc-c++ glibc glibc-devel libstdc++-devel \
  insserv-compat libaio1 libaio-devel libelf1 libgcc_s1 libstdc++6 \
  libcap1 libcap2 libcap-ng0 libcap-ng-utils libcap-progs \
  libpcre1 libpcre16-0 libpng16-16 libjpeg8 libjpeg62 libtiff5 \
  libpcap1 xz pixz mksh systemd-sysvcompat \
  libX11-6 libXau6 libXext6 libXext-devel libXi6 libXi-devel \
  libXrender1 libXrender-devel libXtst6 \
  libaio1-32bit libcap1-32bit libcap2-32bit libcap-ng0-32bit \
  libgcc_s1-32bit libpng16-16-32bit libstdc++6-32bit \
  libXi6-32bit libXrender1-32bit libXtst6-32bit \
  smartmontools sysstat
```

A C compiler is not strictly required to install the base software, but it is required when relinking for an RU. Installing it now avoids a later interruption.

## 5. Create the operating-system account

Create the standard Oracle groups and the `oracle` account:

```bash
getent group oinstall >/dev/null || groupadd -g 54321 oinstall
getent group dba >/dev/null || groupadd -g 54322 dba
getent passwd oracle >/dev/null || \
  useradd -u 54321 -g oinstall -G dba -m -s /bin/bash oracle

usermod -aG dba oracle
passwd oracle
```

The primary group must be `oinstall`, and the account must also be a member of `dba`:

```bash
id oracle
```

Set a strong password when prompted. The simple lab credential used during this installation must not be reused on a production system.

## 6. Configure kernel parameters and shell limits

Calculate shared-memory values from the installed physical memory. The examples below use one half of physical memory for `kernel.shmmax` and convert that value to 4 KB pages for `kernel.shmall`:

```bash
memory_bytes=$(awk '/^MemTotal:/ { printf "%.0f", $2 * 1024 }' /proc/meminfo)
shmmax=$((memory_bytes / 2))
shmall=$((shmmax / 4096))

cat > /etc/sysctl.d/99-oracle.conf <<EOF
fs.aio-max-nr = 1048576
fs.file-max = 6815744
kernel.shmall = ${shmall}
kernel.shmmax = ${shmmax}
kernel.shmmni = 4096
kernel.sem = 250 32000 100 128
net.ipv4.ip_local_port_range = 9000 65500
net.core.rmem_default = 262144
net.core.rmem_max = 4194304
net.core.wmem_default = 262144
net.core.wmem_max = 1048576
EOF

sysctl --system
```

Configure the process and file limits:

```bash
cat > /etc/security/limits.d/99-oracle.conf <<'EOF'
oracle soft nproc 2047
oracle hard nproc 16384
oracle soft nofile 1024
oracle hard nofile 65536
oracle soft stack 10240
oracle hard stack 32768
EOF

grep pam_limits /etc/pam.d/common-session
```

Open a new `oracle` login session and verify the limits:

```bash
su - oracle -c 'ulimit -Sn; ulimit -Hn; ulimit -Su; ulimit -Hu'
```

## 7. Create the directory layout

For this layout, all Oracle-related files are consolidated under `ORACLE_BASE`. The inventory location and directory name are configurable; this guide uses the all-lowercase name `inventory`. OUI reports `INS-32055` as a recommendation when this directory is inside `ORACLE_BASE`, which is acceptable when the catalog and Oracle Base are backed up and managed together.

```bash
install -d -m 755 /data
install -d -m 755 /data/plm
install -d -o oracle -g oinstall -m 775 /data/plm/oracle/inventory
install -d -o oracle -g oinstall -m 775 \
  /data/plm/oracle/product/19.0.0/dbhome_1 \
  /data/plm/oracle/oradata \
  /data/plm/oracle/fast_recovery_area \
  /data/plm/oracle/admin \
  /data/plm/oracle/tmp

chown -R oracle:oinstall /data/plm/oracle
chmod 755 /data/plm
```

The most important paths are:

| Path | Purpose |
| --- | --- |
| `/data/plm/oracle` | `ORACLE_BASE` |
| `/data/plm/oracle/product/19.0.0/dbhome_1` | `ORACLE_HOME` |
| `/data/plm/oracle/inventory` | Central Inventory |
| `/data/plm/oracle/oradata` | CDB and PDB data files |
| `/data/plm/oracle/fast_recovery_area` | Fast Recovery Area |

None of the directories above needs to contain a database data file at this point.

Add the Oracle environment to `/home/oracle/.bash_profile`:

```bash
cat >> /home/oracle/.bash_profile <<'EOF'

export ORACLE_BASE=/data/plm/oracle
export ORACLE_HOME=$ORACLE_BASE/product/19.0.0/dbhome_1
export ORACLE_SID=tcdb
export PATH=$ORACLE_HOME/bin:$PATH
export LD_LIBRARY_PATH=$ORACLE_HOME/lib
EOF
```

Do not set `TNS_ADMIN` unless your environment requires a separate centralized Oracle Net configuration directory.

## 8. Stage and verify the installation media

Copy `LINUX.X64_193000_db_home.zip` to the server, then verify its size and SHA-256 checksum:

```bash
ls -lh /tmp/LINUX.X64_193000_db_home.zip
sha256sum /tmp/LINUX.X64_193000_db_home.zip
```

The media used for this installation has the following checksum:

```text
ba8329c757133da313ed3b6d7f86c5ac42cd9970a28bf2e6233f3235233aa8d8
```

Starting with Oracle 19c, the installation media is extracted directly into an empty `ORACLE_HOME`, and `runInstaller` is executed from that directory:

```bash
su - oracle
umask 022
cd "$ORACLE_HOME"
unzip -q /tmp/LINUX.X64_193000_db_home.zip
```

Confirm that `ORACLE_HOME` contains `runInstaller` and the `install` directory.

## 9. Handle the glibc pthread linker stub

With the newer glibc included in current SLES 15 service packs, the Oracle 19.3 linker may fail with:

```text
/usr/bin/ld: cannot find /usr/lib64/libpthread_nonshared.a: No such file or directory
genclntsh: Failed to link libclntshcore.so.19.1
```

The 19.3 media uses an old linker script at `$ORACLE_HOME/lib/stubs/libpthread.so` that references a file no longer provided by the current glibc package. On affected systems, back up the Oracle stub and remove only the obsolete archive reference:

```bash
stub="$ORACLE_HOME/lib/stubs/libpthread.so"
cp -a "$stub" "${stub}.oracle1930.bak"
sed -i \
  's#GROUP ( libpthread.so.0 /usr/lib64/libpthread_nonshared.a )#GROUP ( libpthread.so.0 )#' \
  "$stub"

cat "$stub"
```

The resulting file should contain:

```text
OUTPUT_FORMAT(elf64-x86-64)
GROUP ( libpthread.so.0 )
```

This is a compatibility workaround for the old 19.3 media. Apply the current certified RU instead where possible.

## 10. Create the silent-install response file

Copy the response file supplied with the installation media:

```bash
cp "$ORACLE_HOME/install/response/db_install.rsp" \
  "$ORACLE_BASE/db_install.rsp"
```

Set at least the following values:

```text
oracle.install.option=INSTALL_DB_SWONLY
UNIX_GROUP_NAME=oinstall
INVENTORY_LOCATION=/data/plm/oracle/inventory
ORACLE_HOME=/data/plm/oracle/product/19.0.0/dbhome_1
ORACLE_BASE=/data/plm/oracle
oracle.install.db.InstallEdition=EE
oracle.install.db.OSDBA_GROUP=dba
oracle.install.db.OSOPER_GROUP=dba
oracle.install.db.OSBACKUPDBA_GROUP=dba
oracle.install.db.OSDGDBA_GROUP=dba
oracle.install.db.OSKMDBA_GROUP=dba
oracle.install.db.OSRACDBA_GROUP=dba
SECURITY_UPDATES_VIA_MYORACLESUPPORT=false
DECLINE_SECURITY_UPDATES=true
```

The last setting is mandatory for a silent offline installation; otherwise OUI can stop at the security-update question.

## 11. Install the Oracle software

Run OUI as the `oracle` user:

```bash
cd "$ORACLE_HOME"
./runInstaller -silent -responseFile "$ORACLE_BASE/db_install.rsp"
```

On the unsupported 19.3 and SLES 15 SP7 combination, OUI can report obsolete package checks such as `libstdc++33`. Review the failed checks in the install log before using the following option:

```bash
./runInstaller -silent \
  -responseFile "$ORACLE_BASE/db_install.rsp" \
  -ignorePrereqFailure
```

Do not ignore unknown missing libraries or permission failures without investigating them. The installation log is stored under `/data/plm/oracle/inventory/logs/`.

When OUI requests root scripts, execute them in the order shown:

```bash
/data/plm/oracle/inventory/orainstRoot.sh
/data/plm/oracle/product/19.0.0/dbhome_1/root.sh
```

After completion, verify that SQL*Plus is available:

```bash
su - oracle -c 'sqlplus -v'
```

The expected version is `19.3.0.0.0` for the base installation.

## 12. Configure Oracle Net

Create `listener.ora` in `$ORACLE_HOME/network/admin`:

```bash
cat > "$ORACLE_HOME/network/admin/listener.ora" <<'EOF'
LISTENER =
  (DESCRIPTION_LIST =
    (DESCRIPTION =
      (ADDRESS = (PROTOCOL = TCP)(HOST = Aurobay-2512)(PORT = 1521))
    )
  )
EOF
```

Replace `Aurobay-2512` with the actual hostname. A listener configuration with two TCP endpoints on the same port can fail with `TNS-12542: address already in use`.

Configure the Teamcenter-compatible SQL*Net expiration setting:

```bash
cat > "$ORACLE_HOME/network/admin/sqlnet.ora" <<'EOF'
SQLNET.EXPIRE_TIME = 10
EOF
```

Start and verify the listener:

```bash
lsnrctl start LISTENER
lsnrctl status LISTENER
```

At this stage, the listener can show `The listener supports no services`. That is normal before a database instance exists or registers with the listener.

## 13. Create the CDB and PDB

The following command creates an empty `tcdb` CDB with one PDB named `tcpdb`. It does not use a Teamcenter-specific template; the Teamcenter schema and tablespaces are normally created later by Deployment Center.

```bash
read -r -s -p "Oracle password: " DB_PASSWORD
echo

dbca -silent -createDatabase \
  -templateName General_Purpose.dbc \
  -gdbName tcdb -sid tcdb \
  -createAsContainerDatabase true \
  -numberOfPDBs 1 \
  -pdbName tcpdb \
  -pdbAdminPassword "$DB_PASSWORD" \
  -sysPassword "$DB_PASSWORD" \
  -systemPassword "$DB_PASSWORD" \
  -characterSet AL32UTF8 \
  -datafileDestination /data/plm/oracle/oradata \
  -recoveryAreaDestination /data/plm/oracle/fast_recovery_area \
  -emConfiguration NONE \
  -sampleSchema false
```

Use separate strong passwords for `SYS`, `SYSTEM`, and the PDB administrator in a production environment. For this isolated lab, the requested simple password was used and accepted with complexity warnings.

Database creation can take several minutes. The main output is written to the terminal, and detailed logs are under `$ORACLE_BASE/cfgtoollogs/dbca/tcdb`.

## 14. Confirm the container and PDB state

Connect with operating-system authentication:

```bash
sqlplus / as sysdba
```

Run the following queries:

```sql
select instance_name, status from v$instance;
select name, open_mode, cdb from v$database;
select name, open_mode from v$pdbs;
select value from nls_database_parameters
where parameter = 'NLS_CHARACTERSET';
```

Expected results include:

```text
tcdb       OPEN
TCDB       READ WRITE   YES
TCPDB      READ WRITE
AL32UTF8
```

If `TCPDB` is mounted, open it and preserve that state across restarts:

```sql
alter pluggable database tcpdb open;
alter pluggable database all save state;
```

Verify a normal password-based connection to the PDB service:

```bash
sqlplus system@//127.0.0.1:1521/tcpdb
```

After entering the password, run:

```sql
show con_name
select name, open_mode from v$database;
```

`CON_NAME` must be `TCPDB`. Applications such as Teamcenter Deployment Center must use the PDB service name `tcpdb`, not only the CDB service name `tcdb`.

## 15. Enable automatic startup with systemd

Set the database entry in `/etc/oratab` to `Y`:

```bash
sed -i '/^tcdb:/d' /etc/oratab
printf 'tcdb:/data/plm/oracle/product/19.0.0/dbhome_1:Y\n' \
  >> /etc/oratab
```

Create the listener service:

```bash
cat > /etc/systemd/system/oracle-listener.service <<'EOF'
[Unit]
Description=Oracle Net listener
After=network-online.target
Wants=network-online.target

[Service]
Type=oneshot
RemainAfterExit=yes
User=oracle
Group=oinstall
Environment=HOME=/home/oracle
Environment=ORACLE_HOME=/data/plm/oracle/product/19.0.0/dbhome_1
Environment=LD_LIBRARY_PATH=/data/plm/oracle/product/19.0.0/dbhome_1/lib
Environment=PATH=/data/plm/oracle/product/19.0.0/dbhome_1/bin:/usr/bin:/bin
ExecStart=/data/plm/oracle/product/19.0.0/dbhome_1/bin/lsnrctl start LISTENER
ExecStop=/data/plm/oracle/product/19.0.0/dbhome_1/bin/lsnrctl stop LISTENER
ExecStopPost=-/bin/sleep 2
TimeoutStartSec=120

[Install]
WantedBy=multi-user.target
EOF
```

Create the database service:

```bash
cat > /etc/systemd/system/oracle-db.service <<'EOF'
[Unit]
Description=Oracle Database instance tcdb
After=network-online.target oracle-listener.service
Wants=network-online.target oracle-listener.service

[Service]
Type=oneshot
RemainAfterExit=yes
User=oracle
Group=oinstall
Environment=HOME=/home/oracle
Environment=ORACLE_BASE=/data/plm/oracle
Environment=ORACLE_HOME=/data/plm/oracle/product/19.0.0/dbhome_1
Environment=ORACLE_SID=tcdb
Environment=LD_LIBRARY_PATH=/data/plm/oracle/product/19.0.0/dbhome_1/lib
Environment=PATH=/data/plm/oracle/product/19.0.0/dbhome_1/bin:/usr/bin:/bin
ExecStart=/data/plm/oracle/product/19.0.0/dbhome_1/bin/dbstart /data/plm/oracle/product/19.0.0/dbhome_1
ExecStop=/data/plm/oracle/product/19.0.0/dbhome_1/bin/dbshut /data/plm/oracle/product/19.0.0/dbhome_1
TimeoutStartSec=600
TimeoutStopSec=300

[Install]
WantedBy=multi-user.target
EOF
```

Enable both services:

```bash
systemctl daemon-reload
systemctl enable oracle-listener oracle-db
```

Because both units use `Type=oneshot` and `RemainAfterExit=yes`, their expected state is `active (exited)`. Their Oracle child processes must still be running.

## 16. Reboot and validate the complete stack

Reboot to verify that the listener, CDB, PDB, and saved PDB state all survive a restart:

```bash
systemctl reboot
```

After the server returns, check the services and port:

```bash
systemctl is-enabled oracle-listener oracle-db
systemctl is-active oracle-listener oracle-db
ss -ltnp | grep 1521
lsnrctl status LISTENER
```

The listener summary should include both services:

```text
Service "tcdb" has 1 instance(s). Instance "tcdb", status READY
Service "tcpdb" has 1 instance(s). Instance "tcdb", status READY
```

Check the database state:

```sql
select instance_name, status from v$instance;
select name, open_mode, cdb from v$database;
select name, open_mode from v$pdbs;
```

The final state should be:

```text
INSTANCE  TCDB OPEN
DATABASE  TCDB READ WRITE YES
PDB       TCPDB READ WRITE
```

Finally, confirm that the database port is reachable from the Teamcenter application server and that `tcpdb` is the service name configured in Deployment Center.

## 17. Troubleshooting

| Symptom | Checks and resolution |
| --- | --- |
| `INS-13013` during OUI prereq checks | Inspect the failed-task section in the install log. On 19.3 and SP7, obsolete checks may remain after the required packages are installed. Review every failure before using `-ignorePrereqFailure`. |
| `libstdc++33` reported as missing | This legacy compatibility package is not present on current SLES 15 SP7. It is a known false positive for the old 19.3 prereq rules. |
| `cannot find /usr/lib64/libpthread_nonshared.a` | Apply the Oracle stub workaround in section 9, or install a current RU that supports the newer glibc package. |
| `INS-32035` or `INS-32047` for Inventory | Use an empty writable Inventory directory owned by `oracle:oinstall`. Do not point a retry at a locked or partially initialized Inventory. |
| `TNS-12542: address already in use` | Use one TCP listener endpoint per host and port, and check that another listener is not already running. |
| `ORA-12541: no listener` | Check `systemctl status oracle-listener`, `lsnrctl status`, the hostname in `listener.ora`, and local firewall rules. |
| `ORA-12514` or no `tcpdb` service | Start the CDB, open the PDB, wait for dynamic registration, or run `alter system register` as `SYSDBA`. |
| `TCPDB` becomes `MOUNTED` after restart | Run `alter pluggable database tcpdb open;` and `alter pluggable database all save state;` while the PDB is open. |
| `TNS-01106: Listener is already running` | The listener is already active. This is not an installation failure. |
| Data files are on NFS | Move them to local storage. Oracle does not support NFS for database data files used by this deployment. |
| Hostname longer than 31 characters | Use a shorter hostname and regenerate the Oracle Net configuration. |

## 18. Production hardening

Before using this database in production:

- Apply a certified 19c RU and confirm the platform remains supported.
- Replace simple lab passwords with strong, unique passwords and rotate them according to policy.
- Restrict TCP port `1521` to the application servers that require database access.
- Place data files and the Fast Recovery Area on a reliable local storage layout.
- Test backup, restore, listener failover, and system restart procedures.
- Review the default password lifetime profile and the customer security baseline.
