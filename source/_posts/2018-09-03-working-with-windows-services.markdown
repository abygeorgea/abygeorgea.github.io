---
layout: post
title: "Working With Windows Services"
date: 2018-09-03 22:59:54 +1000
comments: true
categories: [codesnippets, Windows Services]
keywords: Working with windows services remotely
description: How to start stop windows services
---

Below is code snippet for working with windows service. It helps to find status of service, start , stop and restart as required.

We need to pass in details of windows services name ( as shown i services.msc ) and machine name(should be in same network).

``` csharp

using System.ServiceProcess;


			internal string FindStatus(string service, string server)
        {
            var myService = new ServiceController(service, server);
            log.Info(String.Format("\tStatus of {0} service in {1} is {2}", service, server, myService.Status.ToString()));
            return myService.Status.ToString();
        }

        internal string StopService(string service, string server)
        {
            var myService = new ServiceController(service, server);
            if (myService.Status == ServiceControllerStatus.Running)
            {
                myService.Stop();
                myService.WaitForStatus(ServiceControllerStatus.Stopped);
                log.Info(String.Format("\t{0} service Stopped in {1}. Current Status is {2}", service, server, myService.Status.ToString()));
            }
            return myService.Status.ToString();
        }

        internal string StartService(string service, string server)
        {
            var myService = new ServiceController(service, server);
            if (myService.Status == ServiceControllerStatus.Stopped)
            {
                myService.Start();
                myService.WaitForStatus(ServiceControllerStatus.Running);
                log.Info(String.Format("\t{0} service Started in {1}. Current Status is {2}", service, server, myService.Status.ToString()));
            }
            return myService.Status.ToString();
        }
        
```


  