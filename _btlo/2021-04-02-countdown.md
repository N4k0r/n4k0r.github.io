---
toc: true
toc_sticky: true
layout: btlo
excerpt: "In a race against time, can you investigate a laptop seized by law enforcement to identify if a bomb threat is real or a hoax?"
permalink: /writeups/btlo/countdown

header:
  teaser: "/assets/images/btlo/official/o2geTMCFKgaYvKvokiy3.jpg"
---

![](/assets/images/btlo/official/o2geTMCFKgaYvKvokiy3.jpg)

## Walkthrough

In this investigation, we have two files, a disk image and a text file. By reading it, we learn that the disk image was generated using [Access Data FTK Imager](https://accessdata.com/product-download).

From that same file we also have all information for the first question:  

[![](/assets/images/btlo/countdown/a15ac1c2e02a4f6396feb03549426a86.png){: .align-center}](/assets/images/btlo/countdown/a15ac1c2e02a4f6396feb03549426a86.png)


Verify the Disk Image. Submit SectorCount and MD5.  
**25165824,5c4e94315039f890e839d6992aeb6c58**
{: .notice--success}

### Investigate Autopsy results

Now let's analyze the disk image witn **Autopsy**. To gain some time, a case already exist with provided analysis results:  

[![](/assets/images/btlo/countdown/4de5be29a29a461d98f95c0e78b6f180.png){: .align-center}](/assets/images/btlo/countdown/4de5be29a29a461d98f95c0e78b6f180.png)

Let's see the modules that were used for the analysis to have a better ideas with the artifacts available to us. For that, we can check the ingest modules:  

[![](/assets/images/btlo/countdown/f738d34b1c964f5bb37c7630bc857139.png){: .align-center}](/assets/images/btlo/countdown/f738d34b1c964f5bb37c7630bc857139.png)

The autopsy modules that were used are the following ones:  

[![](/assets/images/btlo/countdown/0efe55099d3644a3a4589e0782a38e7b.png){: .align-center}](/assets/images/btlo/countdown/0efe55099d3644a3a4589e0782a38e7b.png)

To find the decryption key for the online messenger app, I first needed to know what application it was refering to.

I first checked the **Installed Programs** in autopsy results, but I didn't find anything relevant. I then checked the [**Prefecth**](https://digital-forensics.sans.org/media/DFIR-Command-Line.pdf "Prefetch is one source of evidence of a program being run on a system, otherwise known as evidence of execution. Prefetch files are created in the C:\Windows\Prefetch folder when a program is run from a specific location. If that program is run from more than one location, there will be a separate Prefetch file created for each location from which the program ran. Prefetch files are not automatically deleted if the related program is deleted and therefore can be a source of historical information.") directory as per the hints.

From Autopsy we navigate to **C:\Windows\Prefetch** and I noticed two applications that could be good candidates, **Signal** and **Skype**:  

[![](/assets/images/btlo/countdown/7eeec3e1c07747a3b0d7ec5895e14247.png){: .align-center}](/assets/images/btlo/countdown/7eeec3e1c07747a3b0d7ec5895e14247.png)

### Access Signal database

After some google searches about retrieving the decryption key from one of the two applications, I found this [article](https://www.bleepingcomputer.com/news/security/signal-desktop-leaves-message-decryption-key-in-plain-sight/).

It appears that the decryption key is stored in clear text in ***%AppData%\Signal\config.json*** ...

In our case the file is located in ***%AppData%\Roaming\Signal\config.json***:  

[![](/assets/images/btlo/countdown/08dace888544417a8b7db1f8cf22a37a.png){: .align-center}](/assets/images/btlo/countdown/08dace888544417a8b7db1f8cf22a37a.png)

(You can either get the value directly from the Text tab from Autopsy or you can extract the file by right clicking on it.)

What is the decryption key of the online messenger app used by Zerry?  
**c2a0e8d6f0853449cfcf4b75176c277535b3677de1bb59186b32f0dc6ed69998**
{: .notice--success}

Ok, so now that we have a decryption key, we need to use it !
From the same article we learn that the related database is located at **%AppData%\Roaming\Signal\sql\db.sqlite**. Let's extract this database and try to open it. From the **Tools** directory, we have the utility **SQLite Database Browser Portable**.

Since we don't have the passphrase, we need to select **Raw Key**:  

[![](/assets/images/btlo/countdown/4f1981df89324a368d83b66d5f337911.png){: .align-center}](/assets/images/btlo/countdown/4f1981df89324a368d83b66d5f337911.png)

Once selected, you'll notice that the background text change with **0x...**, this means that we need to put 0x at the beginning of the key (you won't be able to paste the key otherwise, took me a while to realise it...):

[![](/assets/images/btlo/countdown/8c6b2f9b4c094b0aa529ede4dfa5e853.png){: .align-center}](/assets/images/btlo/countdown/8c6b2f9b4c094b0aa529ede4dfa5e853.png)

We can then decrypt the database with the default options:  

[![](/assets/images/btlo/countdown/12eb8a6bc6494bc685031bb16def91b0.png){: .align-center}](/assets/images/btlo/countdown/12eb8a6bc6494bc685031bb16def91b0.png)

In the database we find a conversation:  

[![](/assets/images/btlo/countdown/6203dca174314e05836859cfdecef93c.png){: .align-center}](/assets/images/btlo/countdown/6203dca174314e05836859cfdecef93c.png)

From the related metadata, we get more information regarding the user:  

[![](/assets/images/btlo/countdown/de8f8092d5024c829a881352d8ee580b.png){: .align-center}](/assets/images/btlo/countdown/de8f8092d5024c829a881352d8ee580b.png)

What is the registered phone number and profile name of Zerry in the messenger application used?  
**13026482364,ZerryThe🔥**
{: .notice--success}

***Note***: There are a lot of emojis in this challenge, and they are required in the answers. For that I used [this website](https://emojipedia.org).

By reviewing the related messages, we can see two friends (Tom and Jerry!) talking about their plan to bomb a city. We also find an email address:  

[![](/assets/images/btlo/countdown/b7959b412b4d4d449c32264fd8a93638.png){: .align-center}](/assets/images/btlo/countdown/b7959b412b4d4d449c32264fd8a93638.png)

What is the email id found in the chat?  
**eekurk@baybabes.com**
{: .notice--success}

We also learn that an email was sent with an attachment, which was then [erased](https://eraser.heidi.ie/).

### Finding the email attachment

When reviewing the recent documents from Autopsy, we notice a [**.lnk**](https://docs.microsoft.com/en-us/openspecs/windows_protocols/ms-shllink/99e8d0e5-5bc6-4aed-af37-da7f584f832a "Specifies the Shell Link Binary File Format, which contains information that can be used to access another data object. The Shell Link Binary File Format is the format of Windows files with the extension LNK.") file with a strange name. 

According to the previous converstaion, the same "time" emoji was used:  

[![](/assets/images/btlo/countdown/e26f4b0ae60c4183b5a7be06044ee1b2.png){: .align-center}](/assets/images/btlo/countdown/e26f4b0ae60c4183b5a7be06044ee1b2.png)

It is more likely that this shortcut is related to the file attached to the email.

What is the filename(including extension) that is received as an attachment via email?  
**⏳📅.PNG**  
{: .notice--success}

As per the hints, since the file is an image, we can check the [**Thumbcache**](https://www.sans.org/security-resources/posters/windows-forensic-analysis/170/download "Thumbnails of pictures, office documents, and folders exist in a database called the thumbcache. Each user will have their own database based on the thumbnail sizes viewed by the user (small, medium, large, and extra-larger)"). This data is stored in a **.db** file, we can access it directly from the ***File Types*** view and extract it:  

[![](/assets/images/btlo/countdown/26339d45b29c4f849e797e4c8cf955b0.png){: .align-center}](/assets/images/btlo/countdown/26339d45b29c4f849e797e4c8cf955b0.png)

The tool **Thumbcache Viewer** is provided, let's use it !
Once opened, we have multiple results.
if we filter by data size we notice that only 2 images have a size different than zero (if you extract all results, you will only get two files).

If we preview the first one, the image shows a date, with a little sun specifying that the hour is in the morning:  

[![](/assets/images/btlo/countdown/323beb4f24d74ed6aae456c35e8dceb6.png){: .align-center}](/assets/images/btlo/countdown/323beb4f24d74ed6aae456c35e8dceb6.png)

What is the Date and Time of the planned attack?  
**01-02-2021 09:00 AM**
{: .notice--success}

### Locate the attack

Now that we have the time of the attack, we need to know where it will occur !

According to the hints, the location is stored in a Sticky Note. They are stored in a database called **plum.sqlite**. Again, we directly accessed it from the **Databases** file type and extract it.

Inside the database we find a note that seems to be a GPS location but the content is encoded.

```
40 qrterrf 45 zvahgrf 28.6776 frpbaqf A, 73 qrterrf 59 zvahgrf 7.944 frpbaqf J
```

As per the hint, we review the user Tor browsing from the **places.sqlite** database to know more how it was encoded.  

[![](/assets/images/btlo/countdown/e0a032b92bbc4747ad4c79fa42c3f8f6.png){: .align-center}](/assets/images/btlo/countdown/e0a032b92bbc4747ad4c79fa42c3f8f6.png)

We just need to apply the **ROT13** algorithm and we get the GPS coordinates.

What is the GPS location of the blast? The format is the same as found in the evidence . [Hint: Encode(XX Degrees,XX Minutes, XX Seconds)]  
**40 degrees 45 minutes 28.6776 seconds N, 73 degrees 59 minutes 7.944 seconds W**
{: .notice--success}

## Final notes

This investigation was the only one available during the private beta, in which I managed to get the first blood !

I wasn't fast enough with the copy/paste on the official release date to get it back though :p

[![](/assets/images/btlo/countdown/41e5dad237b9456b87ed9535d03f01fb.png){: .align-center}](/assets/images/btlo/countdown/41e5dad237b9456b87ed9535d03f01fb.png)

## Resources

- <https://accessdata.com/product-download>

- <https://www.bleepingcomputer.com/news/security/signal-desktop-leaves-message-decryption-key-in-plain-sight/>

- <https://emojipedia.org/collision/>

- <https://eraser.heidi.ie/>

- <https://docs.microsoft.com/en-us/openspecs/windows_protocols/ms-shllink/99e8d0e5-5bc6-4aed-af37-da7f584f832a >

- <https://www.sans.org/security-resources/posters/windows-forensic-analysis/170/download>

- <https://digital-forensics.sans.org/media/DFIR-Command-Line.pdf>
