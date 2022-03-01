---


---

<h2 id="local-yum-repository-setup-in-centos-7">Local Yum Repository Setup in CentOS 7</h2>
<p>Local yum repository is essential in setting up the cluster environment comprising of tens, hundreds, or thousands of nodes in a cluster. It is not a feasible idea to hit the internet and download the required binaries and packages to be installed in each node of a cluster, because of multiple security and performance reasons.</p>
<p>Instead, setting up a local yum repository copying the required binaries and packages into it, will be helpful to download a install in other cluster nodes locally pointing to local yum repo instead of hitting internet using wget or similar utilities to download the required files.</p>
<p>In this post, we will use cloudera manager server (master node) to configure the local yum repo in it and the other cluster nodes download and install the packages pointing to local yum repo in master node.</p>
<p>Almost all the vendors maintain repository servers and provide .repo file which can be downloaded to <strong>/etc/yum.repos.d</strong>. Packages will be available and when you try to install using yum, it will download files from the internet and then install.</p>
<p>Below required packages are required to be installed to setup the local yum repo.</p>
<pre><code>sudo yum -y install wget createrepo yum-utils
</code></pre>

