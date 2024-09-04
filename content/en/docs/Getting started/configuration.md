---
title: "Configuring your newly deployed Kairos cluster"
linkTitle: "Configuration"
weight: 2
description: |
  Learn the basics of configuring an immutable OS like Kairos
---

{{% alert title="Prerequisites" color="warning" %}}
All you need for this guide is a single node Kairos cluster. If you don't have one yet, you can follow the [Getting Started guide](/docs/getting-started) to deploy one.
{{% /alert %}}

{{% alert title="Objective" %}}
This guide will teach you the basics about immutability and configuration in Kairos. We will achieve this by configuring the hostname of your Kairos node.
{{% /alert %}}

## Prerequisites

- A single node Kairos cluster as the one deployed in the [Getting Started guide](/docs/getting-started).

## Do you prefer to watch a video?

{{< youtube id="Rom4uPatO3k" title="Configuring a Kairos node" >}}

## How is Kairos immutable?

An immutable OS is an operating system that limits the amounts of changes you can do after it is deployed. Full immutability is not possible, as we still need to be able to configure the OS to our needs and to save data for our applications.

Within Kairos you will find two ways in which immutability is accomplished: Read Only Mounts and OverlayFS.

### Read Only Mounts

By default, Kairos is mounted as read-only. If you try installing a package, for example, you will get an error like the following:

```bash
sudo apt update
...
sudo apt install ruby
...
dpkg: error while cleaning up:
 unable to remove newly-extracted version of '/usr/share/doc/libruby': Read-only file system
...
```

### OverlayFS

Alternatively, some directories are mounted as read-write using OverlayFS. This allows you to write to the OS, but the changes are not persistent across reboots.

Do the next experiment as root:

1. Create the file `/etc/hostname`
2. Add the text "master-node" in it and save
3. Reboot the node

After the node has rebooted, you will notice that the file is not present

```bash
cat /etc/hostname
cat: /etc/hostname: No such file or directory
```

## Configuring Kairos

So how do you configure Kairos if it is immutable? The answer is simple: you use configuration files. These files are read at boot time and the changes are applied to the OS on every boot.

Your system is already configured this way. Have a look at the file `/oem/90_custom.yaml`. If you followed the [Getting Started guide](/docs/getting-started), you should see a file with similar content to mine:

```yaml
#cloud-config

install:
    device: /dev/vda
k3s:
    enabled: true
name: Config generated by the installer
stages:
    network:
        - users:
            kairos:
                groups:
                    - admin
                name: kairos
                passwd: kairos
                ssh_authorized_keys:
                    - github:mauromorales
```

The name of the file is not important, but the extension and the location are. You must add your configuration files to `/oem/` and they must have a `.yaml` extension. Finally, the first line of the file must be `#cloud-config` to be recognized by the system.

The `install` directive is used to give instructions on how to install the system when it runs on the first boot, so let's ignore it for now.

The `k3s` directive is used to enable or disable the Kubernetes distribution that comes with Kairos. If you set it to `false`, the K3s services will not start on boot.

The `name` directive can be added at every level of the configuration file. It is useful to distinguish between different steps.

The `stages` directive is used to define the different stages of the configuration. The `network` stage is used to configure the system when network starts. In this case, we create a user called `kairos` with the password `kairos` and an SSH key.

## Configuring the hostname

Configuring the hostname is a simple task. All we have to do is add the `hostname` directive at the root level of the configuration file with the value we want to give to this node. The order of the directives is not important. Let's add it after the `k3s` directive:

```yaml
#cloud-config

install:
    device: /dev/vda
k3s:
    enabled: true
hostname: "master-node"
name: Config generated by the installer
stages:
    network:
        - users:
            kairos:
                groups:
                    - admin
                name: kairos
                passwd: kairos
                ssh_authorized_keys:
                    - github:mauromorales
```

Save the file and reboot the node. After the reboot, you should see the new hostname on the prompt.

```bash
kairos@master-node:~$
```

Let's take this a bit further. In the future we want to add a second master node to our cluster. Let's then use a value that is unique to this node and add it as a suffix.

```yaml
#cloud-config

install:
    device: /dev/vda
k3s:
    enabled: true
hostname: "master-{{ trunc 4 .MachineID }}"
name: Config generated by the installer
stages:
    network:
        - users:
            kairos:
                groups:
                    - admin
                name: kairos
                passwd: kairos
                ssh_authorized_keys:
                    - github:mauromorales
```

{{% alert color="info" %}}
The `.MachineID` comes from [YIP](https://github.com/mudler/yip) which is processing our configuration file, and `trunc` comes from [sprig](http://masterminds.github.io/sprig/) which extends Golang Templating functions.
{{% /alert %}}

Save the file and reboot the node. After the reboot, you should see something similar to the following:

```bash
kairos@master-fb0a:~$
```

## Conclusion

Congrats! You are one step closer to mastering Kairos. In this guide, you learned the basics of immutability and configuration in Kairos. You also learned how to configure the hostname of your Kairos node.

## Frequently Asked Questions (FAQs)

**Why is immutability important?**

It reduces the attack surface of the OS and the chances of having snowflakes. Learn more about [Immutable Linux OS](/blog/2023/03/22/understanding-immutable-linux-os-benefits-architecture-and-challenges/).

**Can I still use my favorite System Configuration Management System?**

Kairos has been designed to be configured a Cloud Init like approach but at the end of the day it is just a Linux distribution. Keep in mind that the immutability of the system will limit the changes you can make, but the better you understand Kairos, the more likely you will be able to configure it to play well with your CMS.

**Why cloud init and not something else?**

Cloud init is a standard in the industry and is widely supported. It is also very flexible and powerful. Read more about [Cloud Init](/docs/architecture/cloud-init/).

<script type="application/ld+json">
{
  "@context": "https://schema.org",
  "@type": "FAQPage",
  "mainEntity": [
    {
      "@type": "Question",
      "name": "Why is immutability important?",
      "acceptedAnswer": {
        "@type": "Answer",
        "text": "It reduces the attack surface of the OS and the chances of having snowflakes. Learn more about [Immutable Linux OS](/blog/2023/03/22/understanding-immutable-linux-os-benefits-architecture-and-challenges/)."
      }
    },
    {
      "@type": "Question",
      "name": "Can I still use my favorite System Configuration Management System?",
      "acceptedAnswer": {
        "@type": "Answer",
        "text": "Kairos has been designed to be configured a Cloud Init like approach but at the end of the day it is just a Linux distribution. Keep in mind that the immutability of the system will limit the changes you can make, but the better you understand Kairos, the more likely you will be able to configure it to play well with your CMS."
      }
    },
    {
      "@type": "Question",
      "name": "Why cloud init and not something else?",
      "acceptedAnswer": {
        "@type": "Answer",
        "text": "Cloud init is a standard in the industry and is widely supported. It is also very flexible and powerful. Read more about [Cloud Init](/docs/architecture/cloud-init/)."
      }
    },
  ]
}
</script>

## What's next?

Ready to upgrade?

<a class="btn btn-lg btn-primary me-3 mb-4" href="{{< relref "../upgrade" >}}">
    Upgrade Guide
</a>

What other configuration options are available?

<a class="btn btn-lg btn-primary me-3 mb-4" href="{{< relref "../Reference/configuration" >}}">
    Configuration Reference
</a>

Learn more about the immutable architecture of Kairos

<a class="btn btn-lg btn-primary me-3 mb-4" href="{{< relref "../Architecture/immutable" >}}">
    Immutable OS
</a>