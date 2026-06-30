---
title: How to start Windchill
description: Detailed steps for starting Windchill(10)
date: 2026-06-30 09:52:51
tags:
  - Stratup
  - Windchill
categories:
  - PLM
  - Windchill
---

## 1 Starting DS

Enter the WindchillDS\sever\bat directory, run start-ds.bat

![Starting DS](./How-to-start-Windchill/StartingDS.png)

You’ll see some outputs that then shut down automatically, whitch is normal.

![DS Output](./How-to-start-Windchill/DSOutput.png)

## 2 Starting the HTTP Server

Enter the {httpserver}\bin directory, run httpd.exe

![Find http.exe](./How-to-start-Windchill/FindHttpd.png)

No output is displayed after it is started.

![httpd](./How-to-start-Windchill/httpd.png)

## 3 Starting the Windchill Method Server

Open the Start menu, all programs, and in the Windchill directory, you can find Windchill Method Server, click and run.

![Find Windchill Method Server script](./How-to-start-Windchill/FindMethodServer.png)

It will start two Java processes, ServerManager and MethodServer, and wait for **MethodServer ready** to appear, indicating successful startup.

![Starting the Method Server](./How-to-start-Windchill/StartingMethodServer.png)

## 4 Testing

Open web browser, enter {Domain name}/Windchill/app

![Test](./How-to-start-Windchill/Test.png)

Maybe these can be configured as automatically satrting service.
