---
layout: post
title: Deploying snappy Ubuntu Core on VMware vSphere
---
## snappy Ubuntu Core and vSphere

With Canonical pushing out new versions of Ubuntu, I wanted to give snappy
Ubuntu Core a try.
Being a small core version of Ubuntu, its suitable for deploying containers and
is easily configured using Cloud-Config by default.

#### Here's how you can deploy Ubuntu Core on vSphere:

Download the OVA image from Ubuntu; OVA images are published using the following scheme:
http://cloud-images.ubuntu.com/ubuntu-core/<release>/core/<channel>/current/core-<channel>-<arch>-cloud.ova

Let's download the latest stable one [here](http://cloud-images.ubuntu.com/ubuntu-core/15.04/core/stable/current/core-stable-amd64-cloud.ova)

**Deploy the OVA file to vSphere:**

- Log on to vSphere
- Select **Deploy OVF Template** and fill in name and location, storage etc.
- Remember to check **Power On After Deployment** and **Finish**

Now, remember that these images are just Ubuntu Snappy Cloud Images wrapped up in a OVA container. So, to get a machine up and running, we have to create a Cloud-Config ISO.

To do so, we need to create two files.  

- First create our files:  
 **cloud-config**
{% highlight yaml %}
#cloud-config
snappy:
    ssh_enabled: true
password: passw0rd
chpasswd: { expire: False }
ssh_pwauth: True
{% endhighlight %}

**meta-data**
{% highlight yaml %}
instance-id: $(uuidgen)
local-hostname: ubuntu-snappy
{% endhighlight %}

- Then create either a ISO or VMDK image:  

{% highlight bash %}
$ dd if=/dev/zero of=bloat_file bs=1M count=10
$ genisoimage \
    -output seed.iso \
    -volid cidata \
    -joliet -rock \
     user-data meta-data bloat_file
$ qemu-img convert -O vmdk seed.iso seed.vmdk
{% endhighlight %}

Upload the ISO or VMDK image to vSphere and attach it to the OVA container you deployed earlier.
Cloud-Init should find the CDROM and read the values. You should now be able to login as "ubuntu" and the password you have set in **cloud-config** and start using snappy Ubuntu Core.

![snappy login]({{ site.baseurl }}/assets/images/snappy.png)
