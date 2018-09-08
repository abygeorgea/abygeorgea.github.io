---
layout: post
title: "Running Command Line from C#"
date: 2016-09-08 23:07:03 +1000
comments: true
categories: [codesnippets, c#, command line]
keywords: How to run command line from C#
description: How to run command line from C#
---

Code Snippet

``` csharp

	private void RunCLIjobsOnLocal(string arguments, int WaitTimePerCommand)
        {
            var psi = new ProcessStartInfo();
            psi.CreateNoWindow = true; //This hides the dos-style black window that the command prompt usually shows
            psi.FileName = @"cmd.exe";
            psi.Arguments = "/C " + arguments;
            psi.RedirectStandardOutput = true;
            psi.RedirectStandardInput = true;
            psi.RedirectStandardError = true;
            psi.UseShellExecute = false;
            var sspw = new SecureString();
            foreach (var c in password)
            {
                sspw.AppendChar(c);
            }
            psi.Domain = domain;
            psi.UserName = userName;
            psi.Password = sspw;
            psi.WorkingDirectory = @"C:\";

            using (Process process = new Process())
            {
                try
                {
                    process.StartInfo = psi;
                    process.Start();
                    var procId = process.Id;
                    string owner = GetProcessOwner(procId);
                    // Synchronously read the standard output of the spawned process. 
                    StreamReader reader = process.StandardOutput;
                    string output = reader.ReadToEnd();
                    reader = process.StandardError;
                    string error = reader.ReadToEnd();
                    if(error.Length >0)
                       process.WaitForExit();
                }
                catch (Exception e)
                {
                    log.Error(e.Message + "\n" + e.StackTrace);
                }
            }
        }

        private string GetProcessOwner(int processId)
        {
            string query = "Select * From Win32_Process Where ProcessID = " + processId;
            ManagementObjectSearcher searcher = new ManagementObjectSearcher(query);
            ManagementObjectCollection processList = searcher.Get();

            foreach (ManagementObject obj in processList)
            {
                string[] argList = new string[] { string.Empty, string.Empty };
                int returnVal = Convert.ToInt32(obj.InvokeMethod("GetOwner", argList));
                if (returnVal == 0)
                {
                   
                    return argList[1] + "\\" + argList[0];
                }
            }
            return "NO OWNER";
        }
        
```

 
 