---
layout: post
title: "Running Command Line in Remote Machine Using WMI"
date: 2018-09-08 21:47:52 +1000
comments: true
categories: [Command Line, WMI, codesnippets]
keywords: Command Line, WMI
description: How to run CMD remotely
---

Recently I had to find a way for running a command line process in server. I had to spend fair bit of time googling for various approaches of doing it. Most of them are by using PSExec.  However there is another approach of using WMI (Windows Management Instrumentation) . Below is one of the approach , which I found at [msdn blog](https://blogs.msdn.microsoft.com/padmanr/2010/05/08/execute-a-process-on-remote-machine-wait-for-it-to-exit-and-retrieve-its-exit-code-using-wmi/).

Below method can be accessed anywhere by 

```
ProcessWMI p = new ProcessWMI();
p.ExecuteRemoteProcessWMI(remoteMachine, sBatFile, timeout);
```

The solution has multiple parts as follows

1. Connect to remote machine using remote machine Name, user name and password
2. Start the remote process. Win32 process and pass the command to be run
3. Find if the remote process is running and if it does, start an event monitor to wait for it to exit
4. Once the process exits, retrieve its exit code


``` Csharp
public class ProcessWMI
{
    public uint ProcessId;
    public int ExitCode;
    public bool EventArrived;
    public ManualResetEvent mre = new ManualResetEvent(false);
    public void ProcessStoptEventArrived(object sender, EventArrivedEventArgs e)
    {
        if ((uint)e.NewEvent.Properties["ProcessId"].Value == ProcessId)
        {
            Console.WriteLine("Process: {0}, Stopped with Code: {1}", (int)(uint)e.NewEvent.Properties["ProcessId"].Value, (int)(uint)e.NewEvent.Properties["ExitStatus"].Value);
            ExitCode = (int)(uint)e.NewEvent.Properties["ExitStatus"].Value;
            EventArrived = true;
            mre.Set();
        }
    }
    public ProcessWMI()
    {
        this.ProcessId = 0;
        ExitCode = -1;
        EventArrived = false;
    }
    public void ExecuteRemoteProcessWMI(string remoteComputerName, string arguments, int WaitTimePerCommand)
    {
        string strUserName = string.Empty;
        try
        {
            ConnectionOptions connOptions = new ConnectionOptions();
            //Note: This will connect  using below credentials. If not provided, it will be based on logged in user
             connOptions.Username = ConfigurationManager.AppSettings["RemoteMachineLogonUser"];
             connOptions.Password = ConfigurationManager.AppSettings["RemoteMachineUserPassword"];
               
            connOptions.Impersonation = ImpersonationLevel.Impersonate;
            connOptions.EnablePrivileges = true;
            ManagementScope manScope = new ManagementScope(String.Format(@"\\{0}\ROOT\CIMV2", remoteComputerName), connOptions);

            try
            {
                manScope.Connect();
            }
            catch (Exception e)
            {
                throw new Exception("Management Connect to remote machine " + remoteComputerName + " as user " + strUserName + " failed with the following error " + e.Message);
            }
            ObjectGetOptions objectGetOptions = new ObjectGetOptions();
            ManagementPath managementPath = new ManagementPath("Win32_Process");
            using (ManagementClass processClass = new ManagementClass(manScope, managementPath, objectGetOptions))
            {
                using (ManagementBaseObject inParams = processClass.GetMethodParameters("Create"))
                {
                    inParams["CommandLine"] = arguments;
                    using (ManagementBaseObject outParams = processClass.InvokeMethod("Create", inParams, null))
                    {

                        if ((uint)outParams["returnValue"] != 0)
                        {
                            throw new Exception("Error while starting process " + arguments + " creation returned an exit code of " + outParams["returnValue"] + ". It was launched as " + strUserName + " on " + remoteComputerName);
                        }
                        this.ProcessId = (uint)outParams["processId"];
                    }
                }
            }

            SelectQuery CheckProcess = new SelectQuery("Select * from Win32_Process Where ProcessId = " + ProcessId);
            using (ManagementObjectSearcher ProcessSearcher = new ManagementObjectSearcher(manScope, CheckProcess))
            {
                using (ManagementObjectCollection MoC = ProcessSearcher.Get())
                {
                    if (MoC.Count == 0)
                    {
                        throw new Exception("ERROR AS WARNING: Process " + arguments + " terminated before it could be tracked on " + remoteComputerName);
                    }
                }
            }

            WqlEventQuery q = new WqlEventQuery("Win32_ProcessStopTrace");
            using (ManagementEventWatcher w = new ManagementEventWatcher(manScope, q))
            {
                w.EventArrived += new EventArrivedEventHandler(this.ProcessStoptEventArrived);
                w.Start();
                if (!mre.WaitOne(WaitTimePerCommand,false))
                {
                    w.Stop();
                    this.EventArrived = false;
                }
                else
                    w.Stop();
            }
            if (!this.EventArrived)
            {
                SelectQuery sq = new SelectQuery("Select * from Win32_Process Where ProcessId = " + ProcessId);
                using (ManagementObjectSearcher searcher = new ManagementObjectSearcher(manScope, sq))
                {
                    foreach (ManagementObject queryObj in searcher.Get())
                    {
                        queryObj.InvokeMethod("Terminate", null);
                        queryObj.Dispose();
                        throw new Exception("Process " + arguments + " timed out and was killed on " + remoteComputerName);
                    }
                }
            }
            else
            {
                if (this.ExitCode != 0)
                    throw new Exception("Process " + arguments + "exited with exit code " + this.ExitCode + " on " + remoteComputerName + " run as " + strUserName);
                else
                    Console.WriteLine("process exited with Exit code 0");
            }
            
        }
        catch (Exception e)
        {
            throw new Exception(string.Format("Execute process failed Machinename {0}, ProcessName {1}, RunAs {2}, Error is {3}, Stack trace {4}", remoteComputerName, arguments, strUserName, e.Message, e.StackTrace), e);
        }
    }
}
```
