---


---

<h2 id="local-yum-repository-setup-in-centos-7">Local Yum Repository Setup in CentOS 7</h2>
<p>Local yum repository is essential in setting up the cluster environment comprising of tens, hundreds, or thousands of nodes in a cluster. It is not a feasible idea to hit the internet and download the required binaries and packages to be installed in each node of a cluster, because of multiple security and performance reasons.</p>
<p>Instead, setting up a local yum repository copying the required binaries and packages into it, will be helpful to download a install in other cluster nodes locally pointing to local yum repo instead of hitting internet using wget or similar utilities to download the required files.</p>
<p>In this post, we will use cloudera manager server (master node) to configure the local yum repo in it and the other cluster nodes download and install the packages pointing to local yum repo in master node.</p>
<p>Almost all the vendors maintain repository servers and provide .repo file which can be downloaded to <strong>/etc/yum.repos.d</strong>. Packages will be available and when you try to install using yum, it will download files from the internet and then install.</p>
<h2 id="install-required-packages">Install required packages:</h2>
<p>Below required packages are required to be installed to setup the local yum repo.</p>
<pre><code>sudo yum -y install wget createrepo yum-utils
</code></pre>
<p>We need Apache webserver to host the cloudera manager package files as local yum repo, so that we need to install httpd, enable the httpd service to start the service on every server startup automatically.</p>
<pre><code>[rvalusa@masternode downloads]$ sudo yum install -y httpd    
[rvalusa@masternode downloads]$ sudo systemctl enable httpd    
[rvalusa@masternode downloads]$ sudo systemctl start httpd    
[rvalusa@masternode downloads]$ systemctl status httpd
httpd.service - The Apache HTTP Server
   Loaded: loaded (/usr/lib/systemd/system/httpd.service; enabled; vendor preset: disabled)
   Active: active (running) since Mon 2022-02-28 02:36:47 AEDT; 4s ago
     Docs: man:httpd(8)
           man:apachectl(8)
 Main PID: 17739 (httpd)
   Status: "Processing requests..."
    Tasks: 6
   CGroup: /system.slice/httpd.service
           ├─17739 /usr/sbin/httpd -DFOREGROUND
           ├─17744 /usr/sbin/httpd -DFOREGROUND
           ├─17745 /usr/sbin/httpd -DFOREGROUND
           ├─17746 /usr/sbin/httpd -DFOREGROUND
           ├─17747 /usr/sbin/httpd -DFOREGROUND
           └─17748 /usr/sbin/httpd -DFOREGROUND

Feb 28 02:36:26 masternode systemd[1]: Starting The Apache HTTP Server...
Feb 28 02:36:36 masternode httpd[17739]: AH00558: httpd: Could not reliably determine the server's fully qualified domain name, using fe80::df17:c835:df0d:386c. Set the 'ServerName' directive globally to suppress this message
Feb 28 02:36:47 masternode systemd[1]: Started The Apache HTTP Server.
</code></pre>
<h2 id="download-cloudera-manager-repo-and-rpm-files">Download cloudera-manager repo and RPM files:</h2>
<pre><code>[rvalusa@masternode downloads]$ cd /etc/yum.repos.d/    
[rvalusa@masternode yum.repos.d]$ sudo wget https://archive.cloudera.com//cm7/7.4.4/redhat7/yum/cloudera-manager.repo
[sudo] password for rvalusa:
--2022-02-28 02:41:41--  https://archive.cloudera.com//cm7/7.4.4/redhat7/yum/cloudera-manager.repo
Resolving archive.cloudera.com (archive.cloudera.com)... 151.101.28.167
Connecting to archive.cloudera.com (archive.cloudera.com)|151.101.28.167|:443... connected.
HTTP request sent, awaiting response... 200 OK
Length: 486 [binary/octet-stream]
Saving to: ‘cloudera-manager.repo’

100%[=======================================================================================================================================================================================================================================================================================&gt;] 486         --.-K/s   in 0s

2022-02-28 02:41:46 (42.1 MB/s) - ‘cloudera-manager.repo’ saved [486/486]

[rvalusa@masternode yum.repos.d]$ ls -ltrah
total 56K
-rw-r--r--.   1 root root  616 Nov 24  2020 CentOS-x86_64-kernel.repo
-rw-r--r--.   1 root root 8.4K Nov 24  2020 CentOS-Vault.repo
-rw-r--r--.   1 root root 1.3K Nov 24  2020 CentOS-Sources.repo
-rw-r--r--.   1 root root  630 Nov 24  2020 CentOS-Media.repo
-rw-r--r--.   1 root root  314 Nov 24  2020 CentOS-fasttrack.repo
-rw-r--r--.   1 root root  649 Nov 24  2020 CentOS-Debuginfo.repo
-rw-r--r--.   1 root root 1.3K Nov 24  2020 CentOS-CR.repo
-rw-r--r--.   1 root root 1.7K Nov 24  2020 CentOS-Base.repo
-rw-r--r--.   1 root root  486 Aug  5  2021 cloudera-manager.repo
drwxr-xr-x. 140 root root 8.0K Feb 28 02:35 ..
drwxr-xr-x.   2 root root  249 Feb 28 02:41 .

[rvalusa@masternode yum.repos.d]$ cat cloudera-manager.repo
[cloudera-manager]
name=Cloudera Manager 7.4.4
baseurl=https://archive.cloudera.com/p/cm7/7.4.4/redhat7/yum/
gpgkey=https://archive.cloudera.com/p/cm7/7.4.4/redhat7/yum/RPM-GPG-KEY-cloudera
username=changeme
password=changeme
gpgcheck=1
enabled=1
autorefresh=0
type=rpm-md

[postgresql10]
name=Postgresql 10
baseurl=https://archive.cloudera.com/postgresql10/redhat7/
gpgkey=https://archive.cloudera.com/postgresql10/redhat7/RPM-GPG-KEY-PGDG-10
enabled=1
gpgcheck=1
module_hotfixes=true
</code></pre>
<p>In the above <strong>cloudera-manager.repo</strong> file remove the <strong>p</strong> from the baseurl and gpgkey of the repo and save it.</p>
<p><strong>before:</strong><br>
baseurl=https://archive.cloudera.com/<strong>p</strong>/cm7/7.4.4/redhat7/yum/<br>
gpgkey=https://archive.cloudera.com/<strong>p</strong>/cm7/7.4.4/redhat7/yum/RPM-GPG-KEY-cloudera</p>
<p><strong>after:</strong><br>
baseurl=https://archive.cloudera.com/cm7/7.4.4/redhat7/yum/<br>
gpgkey=https://archive.cloudera.com/cm7/7.4.4/redhat7/yum/RPM-GPG-KEY-cloudera</p>

