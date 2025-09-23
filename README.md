# The Greenholt Phish room in TryHackMe

## Room link: [The Greenholt Phish](https://tryhackme.com/room/phishingemails5fgjlzxc)

---

Tools used:
- dig command
- mxtoolbox
- Mozilla Thunderbird
- CyberChef
- VirusTotal

In cyber security, the strength of a secure system is measured by its weakest link.

The weakest link is usually the **human element**; you can have the strongest and most up-to-date firewalls, IPS, and antivirus software, but still get compromised due to a phishing link clicked by an 
uneducated employee, or an attachment downloaded by an unaware target whose machine has access to confidential resources.


In this room, we will focus on phishing emails.


**Phishing** is a type of social engineering attacks that attempts to use the human psychology to manipulate victims into giving access of their resources or internal networks to threat actors.

Threat actors try to establish themselves as figures of authority or a close aquintance to the victim to gain his/her trust; they can also pose as a business partner or a customer to try and steal 
financial data or wire money into anonymous accounts or convert it into crypto currencies.

The problem with phishing is that there is no sophisticated tool that can prevent 100% of phishing attempts; with a very elegant and well-written email, you can trick at least 1 employee into 
interacting with your malicious links or attachments, regardless of their technical background.


In this scenario, we will be looking at a sample of a phishing email and analyze it step by step to figure out how exactly it is a phishing email:


The email body is this:

![email body](https://github.com/Aiden971/cybersecurity-writeups/blob/main/Screenshots/The%20Greenholt%20Phish/email%20body.png)


Some emails are very well crafted that you will not be able to tell the signs it is a phishing emails, but let us spot the obvious signs first.

Usually, a legitimate email starts with greeting the person by his/her name; this email greets with a general greeting "Good day webmaster@redacted.org".

**Note**: Some phishing emails can also have a greeting with someone's name, such as spear phishing, because the attacker usually gathers some information about his target before commencing his 
attack.

> Hence, we can say that if the recipient name is mentioned, we need to conduct more checks before determining the email is legitimate;
however, if the recipient name is not mentioned, this is a huge red flag.



The second red flag is the "reply-back" address as you can see below:

![reply-back address](https://github.com/Aiden971/cybersecurity-writeups/blob/main/Screenshots/The%20Greenholt%20Phish/replyback.png)

The reply back address is not the same as the sender, which means 2 things:

1. The sender address was most likely spoofed by an attacker to send on his behalf
2. The attacker has established another email domain quite similar to the sender address to direct all replies to his fraudelant address.


We can also see the sender has put an attachment with his email.

In phishing trainings, we always say:
> "An email is guilty until proven otherwise

This means, do **NOT** click on any links nor download any attachments in any emails until you have confirmed it is legitimate.

> Treat every email as phishing by default until proven otherwise.


Now, let us look into the advanced details that a normal user would not look into.


On Thunderbird, there is a "More" button on when the email is previewed.
Inside it, click on "View source"

![view source](https://github.com/Aiden971/cybersecurity-writeups/blob/main/Screenshots/The%20Greenholt%20Phish/view%20source.png)

This will open up a new window containing the full details of the email, including its headers and the hashed version of the email body.


Among these headers, we have the **return-path** header set to info@mutawamarine.com

This is actually the legitimate domain of the company which the attacker spoofed.

The best header to investigate this spoofing is the **SPF** header.

SPF describes all the hops (email servers) that the email passed in order to reach its destination.

The first ever hop contains the originating IP that sent this email, which is **192.119.71.157**

Using WHOIS, we can figure out the owner of this IP.

![whois](https://github.com/Aiden971/cybersecurity-writeups/blob/main/Screenshots/The%20Greenholt%20Phish/whois.png)

To check the SPF record of the original legitimate domain, we can use online tools or our OS command line.

Head over to the terminal in Linux and type this command:

```
dig TXT mutawamarine.com
```

The following result is generated:

![spf record](https://github.com/Aiden971/cybersecurity-writeups/blob/main/Screenshots/The%20Greenholt%20Phish/spf%20record.png)

The SPF record is **v=spf1 include:spf.protection.outlook.com -all**


If you are asking what this record will be used for, it will determine the policy of dealing with spoofing set by the domain owner.

 > the "-all" flag indicates the spoofs are forbidden.

 Another important header is the DMARC, which is a header describing the policy set by the email administrators should the DMARC check fails.

 To check the DMARC record, we can use mxtoolbox, in which the answer is **v=DMARC1; p=quarantine; fo=1**

![dmarc](https://github.com/Aiden971/cybersecurity-writeups/blob/main/Screenshots/The%20Greenholt%20Phish/dmarc%20record.png)

This indicates the policy is to quarantine the email should it fails the DMARC checks.
<br>

### Now, let's analyze the attachments.

You can check whether this attachment is malicious or not without interacting with it using some tools.

First, the headers indicate the attachment name:

![attachment name](https://github.com/Aiden971/cybersecurity-writeups/blob/main/Screenshots/The%20Greenholt%20Phish/attachment%20name.png)

We need to generate the sha256 hash of this file, so we download it in a secure VM environment, head over to CyberChef tool, set the recipe to hashing, and input the file.

![cyberchef](https://github.com/Aiden971/cybersecurity-writeups/blob/main/Screenshots/The%20Greenholt%20Phish/cyberchef.png)

The resulting hash will be 2e91c533615a9bb8929ac4bb76707b2444597ce063d84a4b33525e25074fff3f


This hash will be useful, because we can copy it to VirusTotal to figure out if this is a popular malware submitted before, which indeed is.

![virustotal](https://github.com/Aiden971/cybersecurity-writeups/blob/main/Screenshots/The%20Greenholt%20Phish/virustotal.png)


From VirusTotal, we can also know the actual size of the attachment, which is 400.26 KB.


And finally, VirusTotal has recorded the actual file extension, which is RAR.



Now we have a complete report of why this email is malicious.
