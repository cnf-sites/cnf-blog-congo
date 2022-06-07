---
title: "Java - Download & Install JDK 1.7 on Windows"
summary: "A detailed step-by-step tutorial on how to download and install jdk 1.7.0_80 on Windows."
url: /java-download-install-jdk-7-windows.html
date: 2017-11-18
lastmod: 2017-11-18
tags: ["posts", "java", "jdk"]
draft: false
---

[Java](https://www.java.com/en/) is a computer programming language that is concurrent, class-based and object-oriented. It was originally developed by James Gosling at Sun Microsystems. Java applications are compiled to bytecode (class file) that can run on any Java virtual machine (JVM) regardless of computer architecture.

Java is currently owned by the Oracle Corporation which acquired Sun Microsystems in 2010. Following tutorial will show you how to setup and configure Java 1.7 on Windows so you can develop and run Java code.

{{< alert "lightbulb" >}}
Check following guides if you are looking to download and install [JDK 1.5](/java-download-install-jdk-5-windows.html), [JDK 1.6](/java-download-install-jdk-6-windows.html), [JDK 1.8](/java-download-install-jdk-8-windows.html), [JDK 1.9](/java-download-install-jdk-9-windows.html) or [JDK 1.10](/java-download-install-jdk-10-windows.html).
{{< /alert >}}

## JDK Download & Install

Java can be obtained from the Oracle Java download page. There are a number of [different Java packages available](https://docs.oracle.com/javaee/6/firstcup/doc/gkhoy.html), for this tutorial we will be installing Java Standard Edition (SE) on Windows.

In order to be able to compile Java code, we need the Java Development Kit (JDK) package that comes with a Java compiler. The JDK package also comes with a Java runtime environment (JRE) that is needed to run compiled Java code.

As we are installing an older Java version, you need to scroll all the way down to the bottom of the [Oracle Java download page](http://www.oracle.com/technetwork/java/javase/downloads/index.html) and click on the `Download` button in the `Java Archive` section. Then look for the `Java SE 7` link and after clicking on it, select the correct operating system under `Java SE Development Kit 7u80`.

> Here is the direct link to [download the jdk 1.7.0_80 installer for Windows 32 or 64 bit](http://www.oracle.com/technetwork/java/javase/downloads/java-archive-downloads-javase7-521261.html).

Accept the License Agreement and pick the correct download for your operating system. In this example, we will use the Windows 64 bit version.

![java 7 download jdk](java-7-download-jdk.png)

Sign in using your Oracle account (or create a new one) and the download should start. Once the download is complete, locate the `jdk-7u80-windows-x64.exe` file and double-click to run the installer.

![java 7 installer start](java-7-installer-start.png)

Click `Next` and on the following screen optionally change the installation location by clicking on the `Change...` button. In this example the install location was changed to `'C:\Java\jdk1.7.0_80'`. From now on we will refer to this directory as: `[java_install_dir]`.

![java 7 jdk location](java-7-jdk-location.png)

Next, the installer will present the installation location of the [public JRE](https://docs.oracle.com/javase/8/docs/technotes/guides/install/windows_jdk_install.html#CHDJCCEG). We will skip this part of the installer as the JDK installed in the previous step comes with a private JRE that can run developed code. Just press `Cancel` and confirm by clicking `Yes` in the popup window.

![java 7 public jre location](java-7-public-jre-location.png)

Click `Next` and then `Close` to finish installing Java.

![java 7 installer finish](java-7-installer-finish.png)

## JDK Configuration

In order for Java applications to be able to run we need to setup a `JAVA_HOME` environment variable that will point to the Java installation directory. In addition, if we want to run Java commands from a command prompt we need to setup the `PATH` environment variable to contain the Java bin directory.

When using Windows the above parameters can be configured on the Environment Variables panel. Click on the `Windows Start` button and enter `env` without quotes as shown below.

![edit environment variables](edit-environment-variables.png)

Environment variables can be set at account level or at system level. For this example click on `Edit environment variables for your account` and following panel should appear.

![environment variables](environment-variables.png)

Click on the `New` button and enter `JAVA_HOME` as variable name and the `[java_install_dir]` as variable value. In this tutorial the installation directory is `C:\Java\jdk1.7.0_80`. Click `OK` to to save.

![java 7 set home](java-7-set-home.png)

Click on the `New` button and enter `PATH` as variable name and `%JAVA_HOME%\bin` as variable value. Click `OK` to save.

> Note that in case a `'PATH'` variable is already present you can add `;%JAVA_HOME%\bin` at the end of the variable value.

![java set path](java-set-path.png)

The result should be as shown below. Click `OK` to close the environment variables panel.

![java 7 environment variables](java-7-environment-variables.png)

In order to test the above configuration, open a command prompt by clicking on the Windows Start button and typing `cmd` followed by pressing `ENTER`. A new command prompt should open in which the following command can be entered to verify the installed Java version:

``` bash
java -version
```

The result should be as shown below.

![java 7 version](java-7-version.png)

This concludes the setting up and configuring JDK 1.7 on Windows.

If you found this post helpful or have any questions or remarks, please leave a comment.
