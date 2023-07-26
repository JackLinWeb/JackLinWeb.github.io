---
title: Send mail from Outlook using SmtpClient
author: jack
date: 2022-11-25 20:55:00 +0800
categories: [blog, csharp]
tags: [csharp]
pin: false
---

From [James](https://dev.to/henryjs)' comment:
**DO NOT USE SMTPCLIENT IN PRODUCTION.** It doesn't support modern protocols and is now obsolete. Use Mailkit or FluentEmail instead.
[Read the Remarks section of Microsoft's docs for more info](https://learn.microsoft.com/en-us/dotnet/api/system.net.mail.smtpclient?view=net-7.0#remarks)

---

Execution environment: Windows 11, .NET 6.0

Create a console application, paste the code in `Program.cs` and replace the following strings:

- sender@outlook.com: The Outlook account to send the letter.
- 123abc!: Outlook account password.
- receiver@gmail.com: The email address of the recipient.

```csharp
using System.Net.Mail;
using System.Net;

var message = new MailMessage();
message.From = new MailAddress("sender@outlook.com");
message.To.Add(new MailAddress("receiver@gmail.com"));
message.IsBodyHtml = true;
message.Subject = "My first smtp email!";
message.Body =
@"
<!DOCTYPE html>
<html>
<body>
    <h3>Hello Jack:</h3>
    <p>This is a Hello World email!</p>
</body>
</html>
";

try
{
    var client = new SmtpClient("smtp-mail.outlook.com", 587);
    client.Credentials = new NetworkCredential("sender@outlook.com", "123abc!");
    client.EnableSsl = true;
    client.Send(message);
}
catch (Exception e)
{
    Console.WriteLine(e);
}
```

Note that the letter may be considered spam the first time it is sent, so it will need to be manually moved to the inbox. The letter looks like the following:


![Image](https://raw.githubusercontent.com/blueskyson/image-host/master/2022/smtpclient.png)
