---
title: "Lost AWS Key Pair: How I Regained Access to My EC2 Instance"
seoTitle: "Regaining Access to Your EC2 Instance: Recovering from a Lost AWS Key "
seoDescription: "Here is how you can regain access to your EC2 instance after losing your AWS key pair. Learn step-by-step methods and best practices to restore connectivity"
datePublished: Mon Jan 22 2024 11:00:39 GMT+0000 (Coordinated Universal Time)
cuid: clrotgkna000009i6g69855qt
slug: lost-aws-key-pair-how-i-regained-access-to-my-ec2-instance
cover: https://cdn.hashnode.com/res/hashnode/image/upload/v1705877748729/05a5636e-eb9a-4ac0-9e4d-f309f47f9c2b.jpeg
ogImage: https://cdn.hashnode.com/res/hashnode/image/upload/v1705877874094/18953df3-5ed7-435e-97b2-942d0abacd33.jpeg
tags: cloud, aws, security, cloud-security

---

## Introduction

In this not-so-brief article, I would like to give a work-through on how I was able to regain access to my AWS instance without having access to the original key pair which was used while creating the instance.

### A Short Story...

I recently started working with a new company, and have been busy making numerous changes to their application. I added new features and fixed a few bugs, and then the time came to deploy the changes on AWS so that users could access them.

Unfortunately, the Ops team informed me that they had no idea where the AWS instance Key Pair was. As an engineer, it was now up to me to figure out how to resolve this issue. Let me take you through the steps you also can use in case if you ever have this same issue.

### What Are Key Pairs?

Now, Imagine you have a special key that can open a secret door. In AWS, key pairs are just like those special keys. You need a key pair when you want to access your AWS resources, like your computer or storage space.

A key pair consists of two parts: a public key and a private key. The public key is like the keyhole in the secret door, and the private key is the unique key that can unlock it.

## Step By Step Guide

Alright. Here's the step-by-step guide you can use if you're also in the kind of fix I was in and need to come up with a solution.

### **Step 1 (Get Instance Information)**

Firstly, you will want to take note of the configuration of the instance you're trying to access. you can do this by logging into your AWS console, navigating to ec2(https://console.aws.amazon.com/ec2/) and selecting the instance you want to connect to. When you've selected the instance, you'll have access to the instance information such as the details, network, storage, root device name etc.

It's essential to take note of this information because you'll need it in the subsequent steps

### **Step 2 (Create a new key pair and launch a new instance)**

To replace your lost key pair, you'll need to create a new one. You have two options to create it; either use the Amazon EC2 console or a third-party tool. If you want to name your new key pair the same as the lost one, you must first delete the existing key pair.

**using the AWS console:**

1. Open the [Amazon EC2 console](https://console.aws.amazon.com/ec2/).
    
2. In the navigation pane, under "Network & Security," look for the "Key Pairs" option.
    
3. Select "Key Pairs" to access the key pair management section (Ensure you are in the correct region and have the necessary permissions to create or manage key pairs.).
    
4. Look for the "Create Key Pair" button and click on it.
    
5. Provide a descriptive name for the key pair.
    
6. Confirm and proceed with the creation.
    

After creating the key pair, you'll need to launch a temporary instance using the new key pairs and then connect to it

### [Step 3:](https://docs.aws.amazon.com/AWSEC2/latest/UserGuide/TroubleshootingInstancesConnecting.html#step-4-launch-temp-instance) (Update Key Pair).

To update the key pair of the original instance with the newly created key pair, you will need to manipulate the volume of the original instance. Firstly, you will have to detach the root volume of the original instance and then attach it to the new instance you just created.

<details data-node-type="hn-details-summary"><summary>Note</summary><div data-type="detailsContent">You will have to temporarily stop the original instance to be able to do this</div></details>

Then, you'll need to add the new public key to the `authorized_keys` of the original volume mounted to the new instance. You can use the following commands to accomplish this:

Connect to the temporary instance.

Attach the volume to the instance and mount it to access the file system. Use the following commands to mount the volume as `/mnt/tempvol` if the device name is `/dev/sdf`.

<details data-node-type="hn-details-summary"><summary>Note</summary><div data-type="detailsContent">The device name might appear differently on your instance. For example, devices mounted as <code>/dev/sdf</code> might show up as <code>/dev/xvdf</code> on the instance. Some versions of Red Hat (or its variants, such as CentOS) might even increment the trailing letter by 4 characters, which <code>/dev/sdf</code> becomes <code>/dev/xvdk</code> .</div></details>

* To check if the volume is partitioned, you can use the `lsblk` command.
    
* ```bash
                  [ec2-user ~]$ lsblk
                  NAME    MAJ:MIN RM  SIZE RO TYPE MOUNTPOINT
                  xvda    202:0    0    8G  0 disk
                  └─xvda1 202:1    0    8G  0 part /
                  xvdf    202:80   0  101G  0 disk
                  └─xvdf1 202:81   0  101G  0 part
                  xvdg    202:96   0   30G  0 disk
    ```
    
    In the preceding example, `/dev/xvda` and `/dev/xvdf` are partitioned volumes, and `/dev/xvdg` are not. If your volume is partitioned, you mount the partition (`/dev/xvdf1)` instead of the raw device (`/dev/xvdf`) in the next steps.
    
* Create a temporary directory to mount the volume.
    
    ```plaintext
    [ec2-user ~]$ sudo mkdir /mnt/tempvol
    ```
    
* To continue, you need to mount the volume or partition at the temporary mount point using the volume name or device name that you previously identified. The command that you should use will depend on the file system that your operating system uses. Please note that the device name may appear differently on your instance. For more information about this, please see the note in Step 6.
    
    * "Amazon Linux, Ubuntu, and Debian" is a list of three different operating systems.
        
        ```plaintext
        [ec2-user ~]$ sudo mount /dev/xvdf1 /mnt/tempvol
        ```
        
    * Amazon Linux 2, CentOS, SUSE Linux 12, and RHEL 7.x
        
        ```plaintext
        [ec2-user ~]$ sudo mount -o nouuid /dev/xvdf1 /mnt/tempvol
        ```
        

<details data-node-type="hn-details-summary"><summary>Note</summary><div data-type="detailsContent">If you encounter an error indicating a corrupt file system, use the <strong>fsck</strong> utility to check and fix any issues.</div></details>

```bash
[ec2-user ~]$ sudo fsck /dev/xvdf1
```

* From the temporary instance, use the following command to update `authorized_keys` on the mounted volume with the new public key from the `authorized_keys` for the temporary instance.
    
    <details data-node-type="hn-details-summary"><summary>Important</summary><div data-type="detailsContent">The username ec2-user is used in these Amazon Linux examples, however, if you are working on Ubuntu instances, you may need to replace it with Ubuntu or another username.</div></details>
    
    ```plaintext
    [ec2-user ~]$ cp .ssh/authorized_keys /mnt/tempvol/home/ec2-user/.ssh/authorized_keys
    ```
    
    If the copy operation is successful, you may proceed to the next step. If not, and you do not have permission to edit files, you must use sudo to update the file. After updating the file, check its permissions to ensure that you can log into the original instance. To check the permissions on the file, use the following command.
    
    ```plaintext
    [ec2-user ~]$ sudo ls -l /mnt/tempvol/home/ec2-user/.ssh
    total 4
    -rw------- 1 222 500 398 Sep 13 22:54 authorized_keys
    ```
    
    In this example, the output `222` is the user ID and `500` the group ID. Next, use **sudo** to re-run the copy command that failed.
    
    ```plaintext
    [ec2-user ~]$ sudo cp .ssh/authorized_keys /mnt/tempvol/home/ec2-user/.ssh/authorized_keys
    ```
    
    Re-run the following command to determine whether the permissions changed.
    
    ```plaintext
    [ec2-user ~]$ sudo ls -l /mnt/tempvol/home/ec2-user/.ssh
    ```
    
    If you have modified the user ID and group ID, use this command to restore them.
    
    ```plaintext
    [ec2-user ~]$ sudo chown 222:500 /mnt/tempvol/home/ec2-user/.ssh/authorized_keyswhile connected to the new (temporary) instance:
    ```
    

Unmount and detach the original volume from the temporary instance after adding the new key pair to the volume, and then reattach it to the original instance.

1. Unmount the volume from the temporary instance to reattach it to the original instance. Use the command "umount /mnt/tempvol".
    
    ```bash
    [ec2-user ~]$ sudo umount /mnt/tempvol
    ```
    
2. To detach the volume from the temporary instance, you need to follow these steps:
    
    * Go to the Amazon EC2 console and click on "Volumes" in the navigation pane.
        
    * Find the root device volume for the original instance (you should have noted its volume ID in a previous step).
        
    * Choose "Actions" and then select "Detach Volume".
        
    * Click on "Detach" to confirm the action.
        
    * Wait for the state of the volume to change to "available". You might need to click on the "Refresh" icon to update the status.
        
3. To reattach the volume to the original instance, follow these steps:
    
    * Select the volume and click on "Actions".
        
    * Choose "Attach Volume".
        
    * From the "Instance" drop-down, select the instance ID of the original instance.
        
    * In the "Device" field, enter the device name that you noted earlier in Step 2 for the original root device attachment ("/dev/sda1" or "/dev/xvda").
        
    * Click on "Attach Volume" to complete the process.
        

<details data-node-type="hn-details-summary"><summary>Important</summary><div data-type="detailsContent">If you don't specify the same device name as the original attachment, you cannot start the original instance. Amazon EC2 expects the root device volume at <code>sda1</code> or <code>/dev/xvda</code>.</div></details>

### Step 4 (connect to the original instance)

Select the original instance and start it after selecting the Instance state. Once the instance has successfully entered the running state, use the private key file for your new key pair to connect to it.

<details data-node-type="hn-details-summary"><summary>Note</summary><div data-type="detailsContent">If you have generated a new key pair with a name different from the original key pair, it is essential to ensure that you specify the name of the new private key file while connecting to your instance. This guarantees you can access your instance using the new key pair and private key file.</div></details>

You can terminate the temporary instance by selecting it and choosing "Terminate the instance" from the Instance state menu.

## Conclusion

There you have it. These were the practical steps that helped me, and I'm confident they can assist you too when faced with the challenge of a missing AWS instance Key Pair. Navigating through such situations requires a systematic and informed approach, and by following the outlined procedures, you can efficiently address the issue and ensure a smooth deployment process.

It's absolutely normal to keep encountering unexpected hurdles while working on any project, but with the right knowledge and a proactive mindset, you can overcome them. Happy deploying!