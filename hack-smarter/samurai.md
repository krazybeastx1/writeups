# Samurai

## Enumeration&#x20;

<details>

<summary>Nmap Scan </summary>

```bash
Nmap scan report for 10.1.55.86
Host is up, received user-set (0.31s latency).
Scanned at 2026-05-10 01:46:18 EDT for 15s

PORT   STATE SERVICE REASON         VERSION
22/tcp open  ssh     syn-ack ttl 62 OpenSSH 8.9p1 Ubuntu 3ubuntu0.13 (Ubuntu Linux; protocol 2.0)
| ssh-hostkey: 
|   256 c3:5a:83:50:80:9a:61:37:05:b7:45:96:cb:ab:1d:1e (ECDSA)
| ecdsa-sha2-nistp256 AAAAE2VjZHNhLXNoYTItbmlzdHAyNTYAAAAIbmlzdHAyNTYAAABBBDnWIbBLcbSbZZmw8nDh5DOA9ecneGMU8Ff1Rm8Frp71DcloANVhYkmErZ3+o839XNGO+k2tmXeNcwJ8jICj06M=
|   256 6b:15:12:60:1b:21:d1:bf:7e:b8:c0:e8:d7:7e:7b:6b (ED25519)
|_ssh-ed25519 AAAAC3NzaC1lZDI1NTE5AAAAIP9JIv57fNRXYSBb4BDtI+WNZG/hfJuGHaaMLL7Iu9PG
80/tcp open  http    syn-ack ttl 62 Apache httpd 2.4.52 ((Ubuntu))
|_http-server-header: Apache/2.4.52 (Ubuntu)
|_http-title: Samurai
|_http-favicon: Unknown favicon MD5: 3E18B73692FF5A74F54EFFB2E047C8CB
| http-methods: 
|_  Supported Methods: GET HEAD POST OPTIONS
Service Info: OS: Linux; CPE: cpe:/o:linux:linux_kernel

```

</details>

* Let's using `feroxbuster` for some directory enumeration&#x20;

<details>

<summary>Feroxbuster </summary>

{% code title="ferox.out" %}
```bash
301      GET        9l       28w      308c http://10.1.55.86/media => http://10.1.55.86/media/
301      GET        9l       28w      309c http://10.1.55.86/images => http://10.1.55.86/images/
301      GET        9l       28w      311c http://10.1.55.86/includes => http://10.1.55.86/includes/
301      GET        9l       28w      312c http://10.1.55.86/templates => http://10.1.55.86/templates/
301      GET        9l       28w      310c http://10.1.55.86/modules => http://10.1.55.86/modules/
301      GET        9l       28w      311c http://10.1.55.86/language => http://10.1.55.86/language/
301      GET        9l       28w      308c http://10.1.55.86/cache => http://10.1.55.86/cache/
301      GET        9l       28w      306c http://10.1.55.86/tmp => http://10.1.55.86/tmp/
301      GET        9l       28w      306c http://10.1.55.86/api => http://10.1.55.86/api/
301      GET        9l       28w      309c http://10.1.55.86/assets => http://10.1.55.86/assets/
301      GET        9l       28w      310c http://10.1.55.86/plugins => http://10.1.55.86/plugins/
301      GET        9l       28w      316c http://10.1.55.86/administrator => http://10.1.55.86/administrator/
301      GET        9l       28w      313c http://10.1.55.86/components => http://10.1.55.86/components/
301      GET        9l       28w      315c http://10.1.55.86/plugins/user => http://10.1.55.86/plugins/user/
301      GET        9l       28w      325c http://10.1.55.86/administrator/language => http://10.1.55.86/administrator/language/
301      GET        9l       28w      322c http://10.1.55.86/administrator/cache => http://10.1.55.86/administrator/cache/
301      GET        9l       28w      321c http://10.1.55.86/administrator/logs => http://10.1.55.86/administrator/logs/
301      GET        9l       28w      318c http://10.1.55.86/plugins/captcha => http://10.1.55.86/plugins/captcha/
301      GET        9l       28w      319c http://10.1.55.86/templates/system => http://10.1.55.86/templates/system/
301      GET        9l       28w      323c http://10.1.55.86/plugins/user/profile => http://10.1.55.86/plugins/user/profile/
301      GET        9l       28w      315c http://10.1.55.86/media/system => http://10.1.55.86/media/system/
301      GET        9l       28w      315c http://10.1.55.86/api/includes => http://10.1.55.86/api/includes/
301      GET        9l       28w      315c http://10.1.55.86/api/language => http://10.1.55.86/api/language/
301      GET        9l       28w      317c http://10.1.55.86/api/components => http://10.1.55.86/api/components/
301      GET        9l       28w      317c http://10.1.55.86/images/banners => http://10.1.55.86/images/banners/
301      GET        9l       28w      325c http://10.1.55.86/administrator/includes => http://10.1.55.86/administrator/includes/
301      GET        9l       28w      324c http://10.1.55.86/administrator/modules => http://10.1.55.86/administrator/modules/
301      GET        9l       28w      321c http://10.1.55.86/administrator/help => http://10.1.55.86/administrator/help/
301      GET        9l       28w      318c http://10.1.55.86/plugins/content => http://10.1.55.86/plugins/content/
301      GET        9l       28w      319c http://10.1.55.86/media/system/css => http://10.1.55.86/media/system/css/
301      GET        9l       28w      322c http://10.1.55.86/media/system/images => http://10.1.55.86/media/system/images/
301      GET        9l       28w      318c http://10.1.55.86/media/system/js => http://10.1.55.86/media/system/js/

```
{% endcode %}

</details>

* Looks like `administrator` is intresting to search lets go with that&#x20;

<figure><img src="../.gitbook/assets/image.png" alt=""><figcaption><p>Joomla Admin Panel </p></figcaption></figure>

* Look like a Joomla server is running , let's try to get a version , so we can look up for any exploits&#x20;

{% embed url="https://github.com/OWASP/joomscan" %}

* Lets use a `joomscan`&#x20;

<pre class="language-bash"><code class="lang-bash"><strong>joomscan -u http://&#x3C;IP>/ -ec 
</strong>
[+] Detecting Joomla Version
[++] Joomla 4.2.5
</code></pre>

{% embed url="https://www.vulncheck.com/blog/joomla-for-rce" %}

* \`C

