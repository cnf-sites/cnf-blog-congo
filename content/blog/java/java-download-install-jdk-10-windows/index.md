---
title: "Java - Download and Install JDK 10 on Windows"
summary: "A detailed step-by-step tutorial on how to download and install jdk 10 on Windows."
url: /java-download-install-jdk-10-windows.html
date: 2018-05-01
lastmod: 2018-05-01
tags: ["posts", "java", "jdk"]
draft: false
---

[Java](https://www.java.com/en/) is a computer programming language that is concurrent, class-based and object-oriented. It was originally developed by James Gosling at Sun Microsystems. Java applications are compiled to bytecode (class file) that can run on any Java virtual machine (JVM) regardless of computer architecture.

Java is currently owned by the Oracle Corporation which acquired Sun Microsystems in 2010. Following tutorial will show you how to setup and configure Java 10 on Windows so you can develop and run Java code.

{{< alert "lightbulb" >}}
Check following guides if you are looking to download and install [JDK 1.5](/java-download-install-jdk-5-windows.html), [JDK 1.6](/java-download-install-jdk-6-windows.html), [JDK 1.7](/java-download-install-jdk-7-windows.html), [JDK 1.8](/java-download-install-jdk-8-windows.html) or [JDK 1.9](/java-download-install-jdk-9-windows.html).
{{< /alert >}}

## JDK Download & Install

Java can be obtained from the Oracle Java download page. There are a number of [different Java packages available](https://docs.oracle.com/javaee/6/firstcup/doc/gkhoy.html), for this tutorial we will be installing Java Standard Edition (SE) on Windows.

In order to be able to compile Java code, we need the Java Development Kit (JDK) package that comes with a Java compiler. The JDK package also comes with a Java runtime environment (JRE) that is needed to run compiled Java code.

As we are installing the latest Java version, you can find it at the top of the [Oracle Java download page](http://www.oracle.com/technetwork/java/javase/downloads/index.html). Click on the `Download` button right below `JDK`.

> Here is the direct link to [download the jdk 10.0.1 installer for Windows 64 bit](http://www.oracle.com/technetwork/java/javase/downloads/jdk10-downloads-4416644.html).

Accept the License Agreement and pick the correct download for your operating system. In this example, we will use the Windows 64 bit version.

![java 10 download jdk](java-10-download-jdk.png)

Sign in using your Oracle account (or create a new one) and the download should start. Once the download is complete, locate the `jdk-10.0.1_windows-x64_bin.exe` file and double-click to run the installer.

![java 10 installer start](java-10-installer-start.png)

Click `Next` and on the following screen optionally change the installation location by clicking on the `Change...` button. In this example the default install location of `'C:\Program Files\Java\jdk-10.0.1\'` was kept. From now on we will refer to this directory as: `[java_install_dir]`.

![java 10 jdk location](java-10-jdk-location.png)

We will not install the public JRE as the JDK Development tools include a private JRE that can run developed code. Select the `Public JRE` dropdown and click on `This feature will not be available.` as shown below.

![java 10 public jre location](java-10-public-jre-location.png)

Click `Next` and then `Close` to finish installing Java.

![java 10 installer finish](java-10-installer-finish.png)

## JDK Configuration

In order for Java applications to be able to run we need to setup a `'JAVA_HOME'` environment variable that will point to the Java installation directory. In addition, if we want to run Java commands from a command prompt we need to setup the `'PATH'` environment variable to contain the Java bin directory.

When using Windows the above parameters can be configured on the Environment Variables panel. Click on the `Windows Start` button and enter `env` without quotes as shown below.

![edit environment variables](edit-environment-variables.png)

Environment variables can be set at account level or at system level. For this example click on `Edit environment variables for your account` and following panel should appear.

![environment variables](environment-variables.png)

Click on the `New` button and enter `JAVA_HOME` as variable name and the `[java_install_dir]` as variable value. In this tutorial the installation directory is `'C:\Program Files\Java\jdk-10.0.1'`. Click `OK` to to save.

![java 10 set home](java-10-set-home.png)

Click on the `New` button and enter `PATH` as variable name and `%JAVA_HOME%\bin` as variable value. Click `OK` to save.

> Note that in case a `'PATH'` variable is already present you can add `;%JAVA_HOME%\bin` at the end of the variable value.

![java set path](java-set-path.png)

The result should be as shown below. Click `OK` to close the environment variables panel.

![java 10 environment variables](java-10-environment-variables.png)

In order to test the above configuration, open a command prompt by clicking on the Windows Start button and typing `cmd` followed by pressing `ENTER`. A new command prompt should open in which the following command can be entered to verify the installed Java version:

``` bash
java -version
```

The result should be as shown below.

![java 10 version](java-10-version.png)

This concludes the setting up and configuring JDK 10 on Windows.

If you found this post helpful or have any questions or remarks, please leave a comment.
