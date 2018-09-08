---
layout: post
title: "Running Powershell Remotely"
date: 2018-09-05 22:46:14 +1000
comments: true
categories: [codesnippets, Powershell]
keywords: Running powershell remotely using C#
description: How to run powershell script on remote machine using C#
---

Code snippet for running power shell on a remote machine. Loosely based on blog post [here](https://com2kid.wordpress.com/2011/09/22/remotely-executing-commands-in-powershell-using-c/) and [here](https://www.codeproject.com/Articles/773685/Enable-Remote-PowerShell-Execution-in-Csharp) 

Add reference to System.Management.Automation

``` csharp

using System.Management.Automation; 
using System.Management.Automation.Runspaces;    


  internal void runPowershellRemotely(string location, string scriptToBeRun)
        {
            string userName = ConfigurationManager.AppSettings["RemoteMachineLogonUser"];
            string password = ConfigurationManager.AppSettings["RemoteMachineUserPassword"]; 
           var securestring = new SecureString();
            foreach (Char c in password){
                securestring.AppendChar(c);
            }
           
            PSCredential creds = new PSCredential(userName, securestring);
            // Remove logging if not needed
            log.Info(String.Format("\tPOWERSHEL : Running Powershell {0} at location {1}", scriptToBeRun, location));
            WSManConnectionInfo connectionInfo = new WSManConnectionInfo();
           
           connectionInfo.ComputerName = ConfigurationManager.AppSettings["RemoteMachine"];
            connectionInfo.Credential = creds;
            Runspace runspace = RunspaceFactory.CreateRunspace(connectionInfo);
            runspace.Open();
            using (PowerShell ps = PowerShell.Create())
            {
                ps.Runspace = runspace;
                ps.AddScript(@"cd "+ location);
                ps.AddScript(scriptToBeRun);
                try
                {
                    var results = ps.Invoke();
                    log.Info("\tPOWERSHEL : Results from Powershell Script is ---------------------------");
                    foreach(var x in results)
                    {
                        log.Info(x.ToString());
                    }
                    log.Info("\tPOWERSHEL : End of results--------------------------------- ---------------------------");
                }
                catch (Exception e)
                {
                    log.Error("\tPOWERSHEL : Exception from running Powershell Script is" + e.ToString());
                }

            }
            runspace.Close();
        }
        
```

