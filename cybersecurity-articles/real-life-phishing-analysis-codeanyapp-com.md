# Real Life Phishing Analysis (codeanyapp.com)

<!-- Original publication date: 2024-02-12 -->

## Introduction

Hello, in this post I am going to analyze a real-life phishing website that I found.

Initially, I was scrolling through Reddit when I found the following:

This is a so-called Smishing message, where the malicious actor uses social-engineering tactics to coerce the recipient into making a transaction. In this case, the malicious actor sends a message impersonating the Romanian Post Office, demanding the user to pay a fee of 1.98 RON (0.43 \$) if he wants his package delivered. In the case that someone could be waiting for their package this can certainly trick them, especially since the Romanian Post Office is sometimes known to send SMS messages.

Moving on, looking at the root domain (*qrco\[.\]de)*, it’s cleary a URL shortener. Quickly going to <https://whois.domaintools.com/qr-code-generator.com> we can see that it probably wasn’t intended for malicious use, as it was registered 4865 days ago:

![](../assets/images/real-life-phishing-analysis-codeanyapp-com-001.png)

Now, moving on, when we click the link we get redirected to *hxxp://servico-secured2-caitlinaloisio91163387\[.\]codeanyapp\[.\]com/posta_romania/POSTA-ROMANA/auth/billing.php*

[**servico-secured2-caitlinaloisio91163387.codeanyapp.com - urlscan.io**<br>*urlscan.io - Website scanner for suspicious and malicious URLs*<span>urlscan.io</span>](https://urlscan.io/result/e08bcda0-2970-4ab9-94fd-06d4c00610f5/ "https://urlscan.io/result/e08bcda0-2970-4ab9-94fd-06d4c00610f5/")

Unfortunately during the the investigation it was quickly taken down, but I managed to gather some basic information:

URL First Hop: *hxxps://qrco\[.\]de/bemy6F*\
Domain First Hop: qrco\[.\]de\
IP First Hop: 18.173.187.98

URL Last Hop: *hxxp://servico-secured2-caitlinaloisio91163387\[.\]codeanyapp\[.\]com/posta_romania/POSTA-ROMANA/auth/billing.php*\
Domain Last Hop: codeanyapp\[.\]com \
IP Last Hop: 45.55.112.74\
Domain created: July 4th 2016, 12:54:28 (UTC)\
Domain registrar: GoDaddy.com, LLC

SHA256 Hashes: [*https://urlscan.io/result/e08bcda0-2970-4ab9-94fd-06d4c00610f5/#indicators*](https://urlscan.io/result/e08bcda0-2970-4ab9-94fd-06d4c00610f5/#indicators)

Additional Phishing Pages: Spanish and South-African Post Office \
Webserver Technology: Apache (Openresty — <https://openresty.org/en/>)\
Admin Panel: “f9.php” -\> PHP File Manager (<https://github.com/alexantr/filemanager>)

## Revenge

Because I was not satisfied and annoyed by the shutdown, I searched in “urlscan.io” for another phishing site on the same domain and found one.

[**swiss-white-pass-tusnaservice805001.codeanyapp.com - urlscan.io**<br>*urlscan.io - Website scanner for suspicious and malicious URLs*<span>urlscan.io</span>](https://urlscan.io/result/e424d301-e45b-417d-8553-575c7b3027f9/ "https://urlscan.io/result/e424d301-e45b-417d-8553-575c7b3027f9/")

By the looks of it, it is yet again a Phishing website for a transportation company. In this case it is for the Swedish company Schweizerische Bundesbahnen. Fortunately, it looks like “urlscan.io” has flagged this URL and also Virustotal (<https://www.virustotal.com/gui/url/3d0b81c3e12cfbe5ca5f27ca6f42f791cdd2671bbf48d8b5b5759089f8bcfb8b?nocache=1>)

Now let’s move onto the real analysis.

## Analysis 1 — Phishing Forms

Using Kali Linux, I accessed the link:

![](../assets/images/real-life-phishing-analysis-codeanyapp-com-002.png)

Looks like this is “index.php”. Let’s try entering some credentials:

user: albert.gelm@gmail.com\
pass: Gelm-Albert99

But first, let’s proxy over to Burpsuite:

![](../assets/images/real-life-phishing-analysis-codeanyapp-com-003.png)

OK, now that we have what we need let’s enter the credentials:

![](../assets/images/real-life-phishing-analysis-codeanyapp-com-004.png)

![](../assets/images/real-life-phishing-analysis-codeanyapp-com-005.png)

We can see in Burpsuite that our email was mapped to “email” and our password to “phone”. We can also see that this calls “send1.php” and we moved from “index.php” to “2.html” (there is no “1.html”).

For now, let’s find on the internet some fake card credentials to keep this going ([<span>www.fakenamegenerator.com</span>](http://www.fakenamegenerator.com)):

Visa 4532 0319 4600 9747 \
Expires 9/2025 \
CVV2 907\
Phone 06211 97 64 73

![](../assets/images/real-life-phishing-analysis-codeanyapp-com-006.png)

Looks like the POST request is for “send2.php”. And we get directed to “3.html”

![](../assets/images/real-life-phishing-analysis-codeanyapp-com-007.png)

Let’s feed it some random number:

![](../assets/images/real-life-phishing-analysis-codeanyapp-com-008.png)

Once again, “send3.php” is called and we get redirected to the official website.

![](../assets/images/real-life-phishing-analysis-codeanyapp-com-009.png)

A summary so far:

1.  index.html (email & password)
2.  send1.php is called
3.  we arrive at 2.html (credit card info)
4.  send2.php is called
5.  we arrive at 3.html (SMS confirmation)
6.  send3.php is called
7.  we arrive at the original website of SwissPass

## Analysis 2 — Digging around

After playing around with the directories of the site, we arrive at the /swiss directory and find multiple folders and a file.

![](../assets/images/real-life-phishing-analysis-codeanyapp-com-010.png)

Looks like \_MACOSX has only remnants of css, js and img. From the folder name, it looks like the malicious actor has a Mac.

The folder Zeos on the other hand is blank.

CHFINAL contains the login pages where we previously were.

But if we look at honeypotbots.dat we get:

![](../assets/images/real-life-phishing-analysis-codeanyapp-com-011.png)

Looks like it was intended as a Honeypot and there is a single repeating IP address.

Moving on, thanks to this video: <https://youtu.be/HwozWl77f3A?si=dD5gP3HZ4xMw57gQ> I was able to come up with the idea to append a “.zip” to /swiss, thus making it “swiss.zip”. As such, we are able to get the original zip file with the code.

In the \_\_MACOSX we can find some more remnants:

![](../assets/images/real-life-phishing-analysis-codeanyapp-com-012.png)

Trying to get some metadata with “exiftool” on one of the hidden files, we get:

![](../assets/images/real-life-phishing-analysis-codeanyapp-com-013.png)

![](../assets/images/real-life-phishing-analysis-codeanyapp-com-014.png)

Most of the files were last modified on: 2023:11:14 12:21:44+02:00 (November 2023)

Most of the files were last opened on: 2024:01:15 05:40:22+02:00 (January 2024)

It also looks like the files were previously in “com.apple.quarantime” and “Microsoft Remote Desktop”. I assume they used the official Microsoft Remote Desktop program to remotely connect to a server.

Let’s move onto the Zeos folder:

![](../assets/images/real-life-phishing-analysis-codeanyapp-com-015.png)

Let’s first open index.php:

![](../assets/images/real-life-phishing-analysis-codeanyapp-com-016.png)

Looks like the original code for the Honeypot, but strangely, when I accessed the index.php previously it didn’t log my correct IP.

![](../assets/images/real-life-phishing-analysis-codeanyapp-com-017.png)

Looks like they have the same “Access Date” but a different “Modification Date” (2022:03:28 15:33:00+03:00).

For all the “anti\*.php” files, looks like they are blocking and redirecting a bunch of IPs and also a lot of web crawlers. I will provide a pastebin link and a Virustotal scan with all the “anti\*.php” file contents:

anti1.php: <https://pastebin.com/2phgy08i>\
<https://www.virustotal.com/gui/file/ef49a8686cc4467154bce98cce6293767e4540e1b2722136fd2f019592f7e446>

anti2.php: <https://pastebin.com/BH74J8UW>\
<https://www.virustotal.com/gui/file/0106a39c2ed9a1cd7e6578208f51ad27329b39b6e3e6a62876c8a204b8ed5aed>

anti3.php: <https://pastebin.com/gXWjHaLJ>\
<https://www.virustotal.com/gui/file/58eed87646b162628a52fd449e16aba3c87163b73177e2bca675410fdc7dd089>

anti4.php: <https://pastebin.com/KLePJB6Z>\
<https://www.virustotal.com/gui/file/e6fd7b76c6eff67bed5be76de27228f48285fb5f460c985e3fe4fd0b9976ce7d>

anti5.php: <https://pastebin.com/98XVGJn5>\
<https://www.virustotal.com/gui/file/f4d4df18152c80b74668e5612a7300418f74dbc8f5950539564dc3e127fff8ab>

anti6.php: there isn’t one

anti7.php: <https://pastebin.com/QPzX18D1>\
<https://www.virustotal.com/gui/file/f24bc0dadb45b054398fc0c38fa138f56dad4f8f40e319b1b8b27f059804db27> (2/60 detections)

anti8.php: <https://pastebin.com/dJYQ7wf1>\
<https://www.virustotal.com/gui/file/28e6e2c9bef4406b6e1b02e98a96881e837568d234a2e4589c15df8912fe24c3>

anti9.php: <https://pastebin.com/0fhWe4e1>\
<https://www.virustotal.com/gui/file/0c97ab9e8d0c75d08830e8016996d2fb139afb762f6362660464d2d50d5913cd> (3/60 detections)

## Analysis 3 — Main Files

Let’s first start with index.php in the CHFINAL directory:

![](../assets/images/real-life-phishing-analysis-codeanyapp-com-018.png)

Looks like this code redirects us to another directory and copies itself over there to create randomness like other login screens. It also creates a “ZA2IR.txt” file to log the users that accessed the initial link, let’s look at it:

![](../assets/images/real-life-phishing-analysis-codeanyapp-com-019.png)

Moving on deeper in the directory we have:

![](../assets/images/real-life-phishing-analysis-codeanyapp-com-020.png)

Let’s start with the “configuration.php” file:

![](../assets/images/real-life-phishing-analysis-codeanyapp-com-021.png)

Looks like this phishing scheme is powered by Telegram and Email. . I assume that the variable “\$email” is not missing in the active site.

Moving onto “send1.php”:

![](../assets/images/real-life-phishing-analysis-codeanyapp-com-022.png)

The credentials are indeed sent through email and also using a Telegram bot to post it to a Telegram group chat.

It also looks like they are mostly the same for the other .php files:

![](../assets/images/real-life-phishing-analysis-codeanyapp-com-023.png)

![](../assets/images/real-life-phishing-analysis-codeanyapp-com-024.png)

![](../assets/images/real-life-phishing-analysis-codeanyapp-com-025.png)

Our only lead from this is the email address:

ana@vps767524.ovh.net

## Analysis 3 — Wordpress

Unfortunately we didn’t get much of a lead, but let’s go to the root domain:

![](../assets/images/real-life-phishing-analysis-codeanyapp-com-026.png)

Looks like he is using WordPress. Let’s try to access /wp-admin:

![](../assets/images/real-life-phishing-analysis-codeanyapp-com-027.png)

Looks like there is a login page. I assume “passswiss” is the username:

![](../assets/images/real-life-phishing-analysis-codeanyapp-com-028.png)

At this point, after knowing the username, let’s try using the “rockyou.txt” wordlist to try and bruteforce the admin panel.

For this task I am going to use Hydra [(https://www.manrajbansal.com/post/how-to-use-hydra-to-brute-force-login-forms](https://www.manrajbansal.com/post/how-to-use-hydra-to-brute-force-login-forms)) and point it to check if the login form is still present in order to determine if the bruteforce was successful.

Unfortunately, after some time it finished and it looks like we weren’t able to find the password.

## Conclusion

Unfortunately we weren’t able to find out a lot. We only know that this was a “mass-produced” phishing site with some protection like IP blocking.

We also saw that they use email and Telegram to receive the credentials.

Lastly, it looks like the password to the admin dashboard isn’t located in the “rockyou.txt” wordlist.
