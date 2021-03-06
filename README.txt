C:\>enum.exe Usage:  enum.exe  [switches]  [hostname|ip]   
The table explains the various options:

Enum.exe Option

Description

-U      Gets userlist

-M      Gets machine list

-N      Gets namelist dump (different from -U|-M)

-S      Gets sharelist

-P      Gets password policy information

-G      Gets group and member list

-L      Gets LSA policy information

-D      Performs a dictionary crack, needs -u and –f

-d      Be detailed, applies to -U and –S

-c      Don’t cancel sessions

-u      Specifies username to use (default "")

-p      Specifies password to use (default "")

-f      Specifies dictfile to use (wants -D)


The first seven options return a wealth of information about the target, provided the IPC$ share is available over port 139 or port 445. By default, it establishes connections over a NULL share—basically, an anonymous user. You can specify all seven options at once, but we’ll break them down a bit to make the output more readable. Combine the –UPG options to gather user-related information:

C:\>enum –UPG 192.168.0.139 server: 192.168.0.139 setting up session... success. password policy:   min length: none   min age: none   max age: 42 days   lockout threshold: none   lockout duration: 30 mins   lockout reset: 30 mins getting user list (pass 1, index 0)... success, got 5.   Administrator  Guest  IUSR_ALPHA  IWAM_ALPHA   TsInternetUser Group: Administrators ALPHA\Administrator Group: Guests ALPHA\Guest ALPHA\TsInternetUser ALPHA\IUSR_ALPHA ALPHA\IWAM_ALPHA Group: Power Users cleaning up... success.
The lines in boldface type suggest that this system would be an excellent target for password guessing. No lockout threshold has been set for incorrect passwords. We also infer from the user list that Internet Information Server (IIS) (IUSR_ALPHA, IWAM_ALPHA) and Terminal Services (TsInternetUser) are installed on the system.

Combine the –MNS options to gather server-related options:

C:\>enum.exe -MNS 10.192.0.139 server: 10.192.0.139 setting up session... success. getting namelist (pass 1)... got 5, 0 left:   Administrator  Guest  IUSR_ALPHA  IWAM_ALPHA   TsInternetUser enumerating shares (pass 1)... got 3 shares, 0 left:   IPC$  ADMIN$  C$ getting machine list (pass 1, index 0)... success, got 0. cleaning up... success.
These options also return a list of users, but they also reveal file shares. In this case, only the default shares are present; however, we can make an educated guess that the system has only one hard drive: C$. Remember that it also had IIS installed. This implies that the web document root is stored on the same drive letter as C:\Winnt\System32. That’s a great combination for us to exploit some IIS-specific issues, such as the Unicode directory traversal vulnerability.

Finally, use the –L option to enumerate the Local Security Authority (LSA) information. This returns data about the system and its relationship to a domain:

C:\>enum.exe -L 10.192.0.139 server: 10.192.0.139 setting up session... success. opening lsa policy... success. server role: 3 [primary (unknown)] names:   netbios: ALPHA   domain: MOONBASE quota:   paged pool limit: 33554432   nonpaged pool limit: 1048576   min work set size: 65536   max work set size: 251658240   pagefile limit: 0   time limit: 0 trusted domains:   indeterminate netlogon done by a PDC server cleaning up... success.
We now know that the system name is ALPHA and it belongs to the domain MOONBASE.

You may often find that the Administrator account has no password. This happens when the administrator flies through the install process and forgets to assign a strong password, or when the administrator assumes that the domain administrator account’s password is strong enough. Use the –u and –p options to specify a particular user’s credentials:

C:\>enum –UMNSPGL –u administrator –p "" 192.168.0.184

