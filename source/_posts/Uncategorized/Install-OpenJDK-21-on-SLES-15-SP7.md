---
title: Install OpenJDK 21 on SLES 15 SP7
description: Install Temurin OpenJDK 21 on SUSE Linux Enterprise Server 15 SP7 and configure a system-wide Java environment.
date: 2026-11-08 21:55:50
tags:
  - Installation
  - JDK
  - Java
  - SLES
categories:
  - Uncategorized
---

OpenJDK 21 is the current long-term support release for many Java applications. This guide installs Temurin OpenJDK 21 from a Linux x64 archive, places it under `/data/plm/jdk/21`, verifies the media checksum, and configures a system-wide Java environment for SLES 15 SP7.

This approach is suitable for Teamcenter and other applications that require a fixed JDK path. It does not depend on the SLES package manager and does not change the system Java alternatives.

## 1. Environment and media

The example uses the following components:

| Item | Value |
| --- | --- |
| Operating system | SLES 15 SP7 x86-64 |
| Distribution | Eclipse Temurin |
| Java version | OpenJDK 21.0.3+9 |
| Install directory | `/data/plm/jdk/21` |
| Archive | `OpenJDK21U-jdk_x64_linux_hotspot_21.0.3_9.tar.gz` |
| Archive size | Approximately 198 MB |
| Installed size | Approximately 346 MB |

OpenJDK 21.0.3 has received later security and maintenance updates. Use the latest supported Temurin 21 release for a production deployment and verify the published checksum before installation.

## 2. Verify the archive

Confirm that the file is a Linux x64 archive:

```bash
file OpenJDK21U-jdk_x64_linux_hotspot_21.0.3_9.tar.gz
```

Inspect the archive root directory before extraction:

```bash
tar -tzf OpenJDK21U-jdk_x64_linux_hotspot_21.0.3_9.tar.gz | head
```

The expected root directory is:

```text
jdk-21.0.3+9/
```

Calculate the SHA-256 checksum:

```bash
sha256sum OpenJDK21U-jdk_x64_linux_hotspot_21.0.3_9.tar.gz
```

The archive used in this installation has the following checksum:

```text
fffa52c22d797b715a962e6c8d11ec7d79b90dd819b5bc51d62137ea4b22a340
```

Compare the result with the checksum published by the download provider. Do not install an archive that fails verification.

## 3. Create the installation directory

Create the parent directory with root ownership and standard read/execute access:

```bash
install -d -m 755 /data/plm/jdk
ls -ld /data/plm/jdk
```

The expected permission is:

```text
drwxr-xr-x root root /data/plm/jdk
```

## 4. Extract OpenJDK

Place the archive in `/tmp`, then extract it into `/data/plm/jdk`:

```bash
tar -xzf /tmp/OpenJDK21U-jdk_x64_linux_hotspot_21.0.3_9.tar.gz \
  -C /data/plm/jdk
```

The archive creates:

```text
/data/plm/jdk/jdk-21.0.3+9
```

Rename it to the stable application path:

```bash
mv /data/plm/jdk/jdk-21.0.3+9 /data/plm/jdk/21
```

Before moving the directory, check that `/data/plm/jdk/21` does not already exist. In a scripted installation, add protection against overwriting an existing JDK:

```bash
test ! -e /data/plm/jdk/21
```

## 5. Set ownership and permissions

Use `root:root` ownership so that application users can use the JDK without being able to modify it:

```bash
chown -R root:root /data/plm/jdk/21
chmod -R u=rwX,go=rX /data/plm/jdk/21
```

Verify the key executables:

```bash
ls -l /data/plm/jdk/21/bin/java
ls -l /data/plm/jdk/21/bin/javac
```

Both files should be executable by the owner, group, and other users.

## 6. Configure the Java environment

Create `/etc/profile.d/jdk.sh` so that new login shells receive the Java environment:

```bash
cat > /etc/profile.d/jdk.sh <<'EOF'
export JAVA_HOME=/data/plm/jdk/21
export JRE_HOME=$JAVA_HOME
export JRE64_HOME=$JAVA_HOME
export PATH=$JAVA_HOME/bin:$PATH
EOF

chown root:root /etc/profile.d/jdk.sh
chmod 644 /etc/profile.d/jdk.sh
```

The resulting file is:

```bash
export JAVA_HOME=/data/plm/jdk/21
export JRE_HOME=$JAVA_HOME
export JRE64_HOME=$JAVA_HOME
export PATH=$JAVA_HOME/bin:$PATH
```

An existing session does not automatically receive the new variables. Reconnect or load the file explicitly:

```bash
source /etc/profile.d/jdk.sh
```

## 7. Verify the installation

Check the configured paths:

```bash
printf 'JAVA_HOME=%s\n' "$JAVA_HOME"
printf 'JRE_HOME=%s\n' "$JRE_HOME"
printf 'JRE64_HOME=%s\n' "$JRE64_HOME"
command -v java
command -v javac
```

The expected paths are:

```text
JAVA_HOME=/data/plm/jdk/21
JRE_HOME=/data/plm/jdk/21
JRE64_HOME=/data/plm/jdk/21
/data/plm/jdk/21/bin/java
/data/plm/jdk/21/bin/javac
```

Verify both the runtime and compiler:

```bash
java -version
javac -version
```

Expected output:

```text
openjdk version "21.0.3" 2024-04-16 LTS
OpenJDK Runtime Environment Temurin-21.0.3+9 (build 21.0.3+9-LTS)
OpenJDK 64-Bit Server VM Temurin-21.0.3+9 (build 21.0.3+9-LTS, mixed mode, sharing)
javac 21.0.3
```

Check the same environment through a new login shell:

```bash
bash -lc 'printenv JAVA_HOME; printenv JRE_HOME; printenv JRE64_HOME; java -version'
```

This confirms that `/etc/profile.d/jdk.sh` is loaded for normal login sessions.

## 8. Upgrade considerations

The `/data/plm/jdk/21` directory is a stable path used by application configuration. When upgrading OpenJDK:

1. Download and verify the new Java 21 archive.
2. Extract it to a temporary version-specific directory.
3. Stop application services that use Java.
4. Preserve the previous JDK until validation is complete.
5. Replace `/data/plm/jdk/21`, or repoint application-specific environment variables to the new version.

Do not replace the active JDK while Java services are starting or while an installer is using it.

## 9. Troubleshooting

| Symptom | Resolution |
| --- | --- |
| `java: command not found` in a new session | Verify that `/etc/profile.d/jdk.sh` exists and the current shell is a login shell or has sourced the file. |
| `JAVA_HOME` is empty | Reconnect, source `/etc/profile.d/jdk.sh`, and check `/etc/profile` loads `/etc/profile.d/*.sh`. |
| `Permission denied` when running Java | Verify that the JDK directory and binary files are readable and executable by the application user. |
| `sha256sum` does not match | Download the archive again from a trusted source and do not install the failed copy. |
| An application still uses an older Java version | Check its service file or startup script for a hard-coded `JAVA_HOME`, `JRE_HOME`, or absolute Java path. |
| Installing a new patch overwrote the active JDK unexpectedly | Use a version-specific directory and switch the stable path only after validation. |
