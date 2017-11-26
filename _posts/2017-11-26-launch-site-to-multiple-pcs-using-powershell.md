---
title: Launch Site to multiple PCs using PowerShell
layout: post
date: '2017-11-26'
excerpt: 'Just about everything you''ll need to style in the theme: headings, paragraphs,
  blockquotes, tables, code blocks, and more.'
tag:
- markdown
- syntax
- sample
- test
- jekyll
---

Using a combination of **PsExec** and **PowerShell**, I created a script to launch mutiple browsers on different laptops. This was a short project I did for a computer lab teacher in my children’s elementary school. This script uses pop ups for data entry.

**Warning**: The password for an admin account is sent in clear text to acheive this.

The script uses 4 main components to achieve this:
<ul>
<li>PsExec, which can be downloaded <a href="https://docs.microsoft.com/en-us/sysinternals/downloads/psexec">here</a>.</li>
<li>PowerShell</li>
<li>Text file with list of PCs</li>
<li>An admin account.</li>
</ul>

Lets get to the good stuff…

## Part 1:

First off, lets create a pop up greeting the user :) I called my program Mass Launch.

{% highlight powershell %}
$wshell = New-Object -ComObject Wscript.Shell
$wshell.Popup("Welcome to Mass Launch! `n`nBy: Dustin Spears",0,"Mass Launch")
The next part of the script will ask for the path to the list of PCs. The last parameter for the input obx pre-fills a suggested location. Then, we end with a if statement, requiring a file to be provided.

$path = Get-Location
[System.Reflection.Assembly]::LoadWithPartialName('Microsoft.VisualBasic') | Out-Null
$file = [Microsoft.VisualBasic.Interaction]::InputBox("Enter the file with computers:", "File Name", "$path\pcs.txt")

If(-Not $file){
    $wshell.Popup("No file was entered.",0,"Mass Launch")
    exit
}
{% endhighlight %}

## Part 2:

Next, we will prompt the user for the website to be sent to the PCs. If the user doesn’t provide a website, the script will end with an error message.

{% highlight powershell %}
[System.Reflection.Assembly]::LoadWithPartialName('Microsoft.VisualBasic') | Out-Null
$site = [Microsoft.VisualBasic.Interaction]::InputBox("Enter the website:", "Website", "$site")

If(-Not $site){
    $wshell.Popup("No site was entered.",0,"Mass Launch")
    exit
}
{% endhighlight %}

## Part 3:

The following part of the script pulls the current domain, and the fills the $pcs array with the computer names. The if statement checks for the pcs to be seperated by 1 space only. The suggested format for the pc text file is as follows: KCSHDB64Z1 KCSG5ZL4Z1 KCS18F54Z1

{% highlight powershell %}
$domain = (Get-WmiObject Win32_ComputerSystem).Name
$pcs = Get-Content $file -Delimiter ' '

If (-Not $pcs) {
    Write-$wshell.Popup("Error! Make sure the computers names in the file are seperated by 1 space only.",0,"Mass Launch")
    exit
}
{% endhighlight %}

## Part 4:

Finally, the list of PCs is iterrated through launching Chrome with the provided site! This part of the script requires the username and password of an administrator account.

{% highlight powershell %}
Try{
    ForEach ($pc In $pcs) {
	$pcTrimmed = $pc.Trim() + "domain"
        Invoke-Expression '& psexec \\$pcTrimmed -u domain\username -p password -i -d "C:\Program Files (x86)\Google\Chrome\Application\chrome.exe" "$site"'
    } 
}
Catch{
    $wshell.Popup("Unexpected error has occured.",0,"Mass Launch")
    exit
}
$wshell.Popup("Thank you for using Mass Launch!.",0,"Mass Launch")
{% endhighlight %}

For those who are just looking for the source code, you may copy from below or download it from my repo. Thanks, and happy scripting!

https://bitbucket.org/snippets/spearsd/Rge8gk/powershell-multi-pc-browser-launcher

## Entire Code:

{% highlight powershell %}
$wshell = New-Object -ComObject Wscript.Shell
$wshell.Popup("Welcome to Mass Launch! `n`nBy: Dustin Spears",0,"Mass Launch")

$path = Get-Location
[System.Reflection.Assembly]::LoadWithPartialName('Microsoft.VisualBasic') | Out-Null
$file = [Microsoft.VisualBasic.Interaction]::InputBox("Enter the file with computers:", "File Name", "$path\pcs.txt")

If(-Not $file){
    $wshell.Popup("No file was entered.",0,"Mass Launch")
    exit
}

[System.Reflection.Assembly]::LoadWithPartialName('Microsoft.VisualBasic') | Out-Null
$site = [Microsoft.VisualBasic.Interaction]::InputBox("Enter the website:", "Website", "$site")

If(-Not $site){
    $wshell.Popup("No site was entered.",0,"Mass Launch")
    exit
}

$domain = (Get-WmiObject Win32_ComputerSystem).Name
$pcs = Get-Content $file -Delimiter ' '

If (-Not $pcs) {
    Write-$wshell.Popup("Error! Make sure the computers names in the file are seperated by 1 space only.",0,"Mass Launch")
    exit
}

Try{
    ForEach ($pc In $pcs) {
	$pcTrimmed = $pc.Trim() + "domain"
        Invoke-Expression '& psexec \\$pcTrimmed -u domain\username -p password -i -d "C:\Program Files (x86)\Google\Chrome\Application\chrome.exe" "$site"'
    } 
}
Catch{
    $wshell.Popup("Unexpected error has occured.",0,"Mass Launch")
    exit
}

$wshell.Popup("Thank you for using Mass Launch!.",0,"Mass Launch")
{% endhighlight %}