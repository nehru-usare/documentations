# ⚙️ Java 21 Setup and Development Environment Guide

Before writing any Java code, you must correctly configure your **Java 21 development environment**.  
A proper setup ensures that compilation, build automation, and project management all work seamlessly — across any operating system.

This guide walks you through **installing, configuring, and verifying Java 21 (LTS)** for a modern developer workflow.

---

## 🧭 Table of Contents

1. [Installing Java 21 (OpenJDK / Oracle JDK)](#1-installing-java-21-openjdk--oracle-jdk)
2. [Setting Environment Variables](#2-setting-environment-variables)
3. [Verifying Installation](#3-verifying-installation)
4. [Configuring Your IDE](#4-configuring-your-ide)
5. [Setting up Maven and Gradle](#5-setting-up-maven-and-gradle)
6. [Checking JDK Tools](#6-checking-jdk-tools)
7. [Troubleshooting Common Issues](#7-troubleshooting-common-issues)
8. [Summary](#8-summary)

---

## 1️⃣ Installing Java 21 (OpenJDK / Oracle JDK)

Java 21 (LTS) can be installed from several reliable vendors.  
You can use **Oracle JDK**, **OpenJDK**, **Eclipse Adoptium**, or **Amazon Corretto**.

### 🔹 Option 1: Using Oracle JDK
👉 [Download from Oracle’s official site](https://www.oracle.com/java/technologies/downloads/)

1. Choose **Java 21 LTS**
2. Select your OS (Windows, macOS, Linux)
3. Install the package (e.g. `.msi` on Windows or `.pkg` on macOS)

---

### 🔹 Option 2: Using OpenJDK (Recommended for Open Source)

#### 🪟 On Windows (via Adoptium)
```powershell
winget install EclipseAdoptium.Temurin.21.JDK
````

#### 🐧 On Ubuntu / Debian

```bash
sudo apt update
sudo apt install openjdk-21-jdk -y
```

#### 🍎 On macOS (via Homebrew)

```bash
brew install openjdk@21
sudo ln -sfn $(brew --prefix openjdk@21)/libexec/openjdk.jdk /Library/Java/JavaVirtualMachines/openjdk-21.jdk
```

---

### 🔹 Option 3: Using Amazon Corretto (Production Grade)

```bash
# Linux
sudo yum install java-21-amazon-corretto-devel -y

# macOS
brew install corretto
```

✅ Corretto is 100% compatible with OpenJDK and production-hardened by AWS.

---

## 2️⃣ Setting Environment Variables

Once installed, Java must be added to your **PATH** and **JAVA_HOME**.

### 🪟 On Windows

1. Open **Environment Variables**
   `Control Panel → System → Advanced → Environment Variables`

2. Add:

   ```
   JAVA_HOME = C:\Program Files\Java\jdk-21
   ```

3. Edit **Path** variable → Add:

   ```
   %JAVA_HOME%\bin
   ```

4. Reopen Command Prompt:

   ```cmd
   echo %JAVA_HOME%
   java -version
   ```

---

### 🐧 On Linux / macOS

Edit `~/.bashrc` or `~/.zshrc`:

```bash
export JAVA_HOME=/usr/lib/jvm/java-21-openjdk-amd64
export PATH=$JAVA_HOME/bin:$PATH
```

Apply changes:

```bash
source ~/.bashrc
```

Check version:

```bash
java -version
```

Output should look like:

```
openjdk version "21" 2023-09-19 LTS
OpenJDK Runtime Environment (build 21+35-LTS)
OpenJDK 64-Bit Server VM (build 21+35-LTS, mixed mode, sharing)
```

---

## 3️⃣ Verifying Installation

Run these commands to confirm:

```bash
java -version
javac -version
```

Both should print version **21.x.x**.

✅ **If you see version mismatch**, it usually means:

* Multiple JDKs installed
* PATH is pointing to older version

Use:

```bash
where java   # Windows
which java   # Linux/Mac
```

to locate the active JDK binary.

---

## 4️⃣ Configuring Your IDE

### 🔹 IntelliJ IDEA (Recommended)

1. Open **File → Project Structure → SDKs**
2. Add **JDK 21**
3. Select it under **Project SDK**
4. Enable:

   * Project Language Level: `21 (Preview features enabled)`
   * Build System: **Maven / Gradle**

### 🔹 VS Code (Lightweight Alternative)

1. Install extensions:

   * `Extension Pack for Java`
   * `Debugger for Java`
   * `Test Runner for Java`
2. Configure settings:

   ```json
   {
     "java.configuration.runtimes": [
       {
         "name": "JavaSE-21",
         "path": "C:\\Program Files\\Java\\jdk-21",
         "default": true
       }
     ]
   }
   ```

### 🔹 Eclipse IDE

* Go to: `Window → Preferences → Java → Installed JREs`
* Add your Java 21 path.

---

## 5️⃣ Setting up Maven and Gradle

### ⚙️ Maven Setup

Download from: [https://maven.apache.org/download.cgi](https://maven.apache.org/download.cgi)

Unzip and set:

```bash
MAVEN_HOME=C:\apache-maven-3.9.6
PATH=%MAVEN_HOME%\bin
```

Verify:

```bash
mvn -version
```

Example output:

```
Apache Maven 3.9.6
Java version: 21
```

---

### ⚙️ Gradle Setup

Install via:

```bash
# Windows
winget install Gradle.Gradle

# Linux
sudo apt install gradle -y
```

Verify:

```bash
gradle -v
```

Gradle integrates tightly with Java 21 for modern builds.

---

## 6️⃣ Checking JDK Tools

Java 21 includes a suite of **developer tools** under `$JAVA_HOME/bin`.

| Tool             | Description                        |
| ---------------- | ---------------------------------- |
| `javac`          | Java compiler                      |
| `java`           | Executes programs                  |
| `javadoc`        | Generates documentation            |
| `jar`            | Create/Extract JAR archives        |
| `jlink`          | Custom runtime image creation      |
| `jpackage`       | Build native installers            |
| `jdeps`          | Analyze class dependencies         |
| `jcmd`           | Send commands to running JVM       |
| `jmap`, `jstack` | Memory and thread dumps            |
| `jfr`            | Java Flight Recorder for profiling |

✅ Check all available tools:

```bash
ls $JAVA_HOME/bin
```

---

## 7️⃣ Troubleshooting Common Issues

| Problem                   | Cause                   | Solution                                  |
| ------------------------- | ----------------------- | ----------------------------------------- |
| `java: command not found` | PATH not set            | Add `JAVA_HOME/bin` to PATH               |
| Version mismatch          | Multiple JDKs installed | Use `where java` / `which java` to verify |
| IDE not detecting JDK     | Wrong SDK path          | Configure JDK 21 manually in IDE          |
| Slow compilation          | Outdated Maven/Gradle   | Update to latest versions                 |
| Permission denied         | Insufficient privileges | Run terminal/IDE as admin or with `sudo`  |

---

## 8️⃣ Summary

✅ You now have:

* Installed **Java 21 LTS**
* Configured environment variables (`JAVA_HOME`, `PATH`)
* Verified setup via CLI
* Integrated **IntelliJ / VS Code**
* Installed **Maven** and **Gradle**
* Learned core **JDK tools**

This setup is **enterprise-ready** — the same used by professional teams building production Java applications.

---

> 🧭 **Next Topic:** [04-java-basics-and-syntax.md → Java 21 Syntax, Keywords, and Language Fundamentals](./04-java-basics-and-syntax.md)