---
layout: post
title: "Sending Emails through SMTP in C#"
date: 2019-04-09 22:03:26 +1000
comments: true
categories: codesnippets
keywords: [Email, C#, codesnippets]
description: How to send emails in C# using SMTP
---

Below is a code snippet for sending emails through C#

``` 

using System;
using System.Collections.Generic;
using System.IO;
using System.Net.Mail;
using System.Text.RegularExpressions;

namespace TestAutomationFramework.Common
{
    public class EmailHelper
    {
        public static void Send(string host, EmailRequest request)
        {
            ValidEmailRequest(request);

            using (var mailMessage = new MailMessage())
            {
                mailMessage.Subject = request.Subject;
                mailMessage.Body = request.Body;
                mailMessage.IsBodyHtml = request.IsBodyHtml;
                mailMessage.From = new MailAddress(request.From);

                request.To.ForEach(t => mailMessage.To.Add(t));
                request.Cc.ForEach(c => mailMessage.CC.Add(c));
                request.Bcc.ForEach(b => mailMessage.Bcc.Add(b));
                request.Attachments.ForEach(a => mailMessage.Attachments.Add(new Attachment(a)));

                var smtpClient = new SmtpClient { Host = host };

                smtpClient.Send(mailMessage);
            }
        }

        public static void ValidEmailRequest(EmailRequest emailRequest)
        {
            if (!IsValidEmailAddress(emailRequest.From))
                throw new Exception($"Email request From:{emailRequest.From} is not a valid email address.");

            emailRequest.To?.ForEach(e =>
            {
                if (!IsValidEmailAddress(e))
                {
                    throw new Exception($"Email request To:{e} is not a valid email address.");
                }
            });

            emailRequest.Cc?.ForEach(e =>
            {
                if (!IsValidEmailAddress(e))
                {
                    throw new Exception($"Email request Cc:{e} is not a valid email address.");
                }
            });

            emailRequest.Bcc?.ForEach(e =>
            {
                if (!IsValidEmailAddress(e))
                {
                    throw new Exception($"Email request Bcc:{e} is not a valid email address.");
                }
            });

            emailRequest.Attachments?.ForEach(a =>
            {
                if (!IsValidAttachment(a))
                {
                    throw new Exception($"Email request Attachment:{a} is not accessible.");
                }
            });
        }

        private static bool IsValidAttachment(string attachment)
        {
            return File.Exists(attachment);
        }

        private static bool IsValidEmailAddress(string emailAddress)
        {
            if (string.IsNullOrWhiteSpace(emailAddress))
                return false;

            //https://msdn.microsoft.com/en-us/library/01escwtf(v=vs.110).aspx
            try
            {
                return Regex.IsMatch(emailAddress,
                      @"^(?("")("".+?(?<!\\)""@)|(([0-9a-z]((\.(?!\.))|[-!#\$%&'\*\+/=\?\^`\{\}\|~\w])*)(?<=[0-9a-z])@))" +
                      @"(?(\[)(\[(\d{1,3}\.){3}\d{1,3}\])|(([0-9a-z][-\w]*[0-9a-z]*\.)+[a-z0-9][\-a-z0-9]{0,22}[a-z0-9]))$",
                      RegexOptions.IgnoreCase, TimeSpan.FromMilliseconds(250));
            }
            catch (RegexMatchTimeoutException)
            {
                return false;
            }
        }
    }

    public class EmailRequest
    {
        public string Subject { get; set; }
        public string Body { get; set; }
        public bool IsBodyHtml { get; set; }
        public string From { get; set; }
        public List<string> To { get; set; }
        public List<string> Cc { get; set; }
        public List<string> Bcc { get; set; }
        public List<string> Attachments { get; set; }
    }

}


```