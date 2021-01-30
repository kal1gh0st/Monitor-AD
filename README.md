# Monitor-AD
The best way is just to double click on PingCastle.exe

This run the program in a mode called the “interactive mode”.

PingCastle.exe --healthcheck --server mydomain.com

The program can be run using a command line. A command line can be run by searching for “cmd” or “command line” in the start menu.

Then a drag and drop of the file “PingCastle.exe” automatically populates the command line with the binary. The same can be done with other files ending with “.exe”.

Additional options can be provided. We will use the following marking for commands:
PingCastle something you have to type.

PingCastle.exe --help

PingCastle can collect logs with the –log switch

However when a command line argument is submitted, the interactive mode is disabled and the module has to be launched manually. To avoid that, the “interactive mode” can be activated manually using the command:

PingCastle.exe --log --interactive

The report can be generated in the interactive mode by choosing “healthcheck” or just by pressing Enter. Indeed it is the default analysis mode.

It can be run using the command:

PingCastle.exe --healthcheck --server mydomain.com

When having existing health check reports

The map can be generated in the interactive mode by choosing “conso”. This mode performs the consolidation report and build the maps.
It can be run using the command:

PingCastle.exe --hc-conso

When no health check report is available

The map can be generated in the interactive mode by choosing “carto”. This mode performs only the part needed for building the map in the health check process to all the available domains. This mode cannot combine existing health check reports.
It can be run using the command:

PingCastle.exe --carto

There are two kinds of map. The first one is the most complete but can be difficult to read. To avoid this difficulty, a simplified most exists where a domain is connected to others only using a single trust. It builds a hierarchy.
When available, the Active Directory health check score is displayed and colored by on it. Trusts are also colored based on the SID Filtering state.

Algorithm to build the graph
PingCastle is using the data included in the report from the most reliable source to the less reliable source.

The most reliable source is domain where the report has been generated.
Then the tool is using direct trust data.
Then the tool is using forest trust information. This information is located in the msDS-TrustForestTrustInfo attribute of a forest trust and in the partition element of the configuration binding context.
The tool is using the information provided by the domain locator service when examining trusts. This information can add the Netbios name or the forest name of a trusted domain.
If the “reachable” option has been set when producing a report, the tool is using domain SID found (in foreign security principals or sid history) to query the domain locator service and guess forest trusts.

1. Get reports
1.1. Option 1: Each domain run PingCastle
PingCastle can be run on every domain of a company using the command:

PingCastle --healthcheck
1.1. Option 2: PingCastle is run at key location
PingCastle can be run on a Bastion Active Directory, generally used to perform administration tasks. In this case, all the domains will be scanned:

PingCastle --healthcheck --server *
The program can be run on every forest root and be limited to that perimeter

PingCastle --healthcheck --server *.forest.root
The tool can be run on every forest child and explore the child and its trusted domains. In this case the forest root is excluded.

PingCastle --healthcheck --explore-trust --server child.forest.root
PingCastle can explore all the domains of all the trusted forests from another forest. This is useful when the root and child doesn’t share the same name.

PingCastle --healthcheck --explore-forest-trust --server anotherforest.root
If needed, exceptions can be set to not scan domains. For example to not scan the Bastion domain multiple times. In this case use the option –explore-exception <domains> where domains are comma separated domain name.

2. Schedule
Even if the management reporting is done on a monthly basis, we recommend to setup a scheduled task on a weekly basis.

This frequency is justified to:

See the improvement almost in real time and avoid the tunnel effect
Detect newly created trusts and be able to remove them if needed with a limited business impact.
Daily scans are not recommended as the additional energy needed to follow up will not provide any additional benefits.

3. Collect the reports
3.1. Encryption to use unsafe channels
Sometimes, domains are unconnected or it is not possible to make the schedule tasks centralize in a single share all the reports. To deal with this case, PingCastle can encrypt the reports to send them in an unsafe channel.

A RSA key pair need to be generated and the public key needs to be shared with all the instance of the program. When producing risks reports and generating the .xml files, add the flag –encrypt to perform the encryption.

You can generate a keypair using the following command and copy the public key in the .config file to be deployed.

PingCastle.exe --generate-key

Starting the task: Generate Key
 Public Key (used on the encryption side):
 <encryptionSettings encryptionKey="default">
 <RSAKeys>
 <!-- encryption key -->
 <KeySettings name="default" publicKey="&lt;RSAKeyValue&gt;&lt;Modulus&gt;h
 4smrLAZZ30QwWXHcT1oNz3hH3Ax2R9T75DlioGFCIdLb0QhUn3N8NWgJ2ZgyUNXn4qU1b0DslOIhK+Cq
 oqCPvXuHjK6TGrMyphtcbZvvgbLxfyalJemczx1+pOuBlqqVdalE94rnnnBr761WIJJnkJdZ0rzYsebn
 DwGuk9kiw8=&lt;/Modulus&gt;&lt;Exponent&gt;AQAB&lt;/Exponent&gt;&lt;/RSAKeyValue
 &gt;"/>
 <!-- end -->
 </RSAKeys>
 </encryptionSettings>
 Private Key (used on the decryption side):
 <encryptionSettings encryptionKey="default">
 <RSAKeys>
 <!-- decryption key -->
 <KeySettings name="39b5d076-17be-4999-b43e-b894a55446a1" privateKey="&lt;R
 SAKeyValue&gt;&lt;Modulus&gt;h4smrLAZZ30QwWXHcT1oNz3hH3Ax2R9T75DlioGFCIdLb0QhUn3
 N8NWgJ2ZgyUNXn4qU1b0DslOIhK+CqoqCPvXuHjK6TGrMyphtcbZvvgbLxfyalJemczx1+pOuBlqqVda
 lE94rnnnBr761WIJJnkJdZ0rzYsebnDwGuk9kiw8=&lt;/Modulus&gt;&lt;Exponent&gt;AQAB&lt
 ;/Exponent&gt;&lt;P&gt;uwgX794pe7O3vIiQR5v03WK3Ug5LUAbXpPF6Xq4qGb3TGprZaJQq5rZ2u
 J4qwRanOa5pI/zv7RhG/4ItesBuAw==&lt;/P&gt;&lt;Q&gt;uYaNLEp9Vh8F29tSH+M4z+OjxPl+UL
 LRjLrssFLTTNsdnrHgAtdJ1lxfIm/gTUa0qPLa9Y/xkUb1khK/+tV3BQ==&lt;/Q&gt;&lt;DP&gt;Fd
 feI8+IfMACh2xTnWljca+jxVuSBCioasUhC4m/tP3sd8D5/zK+x+8rcmhWifKBWUU7Vk6mHsSlFhY4BY
 wPzQ==&lt;/DP&gt;&lt;DQ&gt;gzfwh8AT0CLXEP6ZomYi257lST8xoUAoyEG5gKjEPJrJ42Fp0HiXB
 9+Dhibc3atBwjEqvv5VXGx06iEK2g27RQ==&lt;/DQ&gt;&lt;InverseQ&gt;HRKFjYwrXqgO4v8Q+J
 SOqR6lSvQ15Z6V4AE23i4xfeuIYWwVf0t8AwgkDfFRQnEyh24byuh5PPzUbDOsUY+eYg==&lt;/Inver
 seQ&gt;&lt;D&gt;QQ6pIXnkt6dvw2P2toOi4eDxjQVs56oBv5rske5YzB8kNeOdmtqHXnEqzb519iQ8
 incZuP1gKNevTwBu1yxkFuFh0dzjS3iBjHvYGtDo5mARiZ1nN8QNI2zKE+Q6qXF8Z+wN3Fv3oBDQXATI
 6IQbgkAxLTMo4CUmtUQ6GvjwFwE=&lt;/D&gt;&lt;/RSAKeyValue&gt;"/>
 <!-- end -->
 </RSAKeys>
 </encryptionSettings>
 Done
 Task Generate Key completed

Then copy the private key section in the PingCastle and PingCastleReporting configuration file (.config) used to consolidate the results. PingCastle will perform the decryption automatically.

The program can generate an encrypted copy of a report (public key needed) and a decrypted copy of a report (private key needed) using the following commands:

PingCastle --reload-report report.xml --encrypt

PingCastle --reload-report encrypted-report.xml

Note: Only one key can be specified for encryption but multiple keys can be used for decryption. Their selection is automatic.

3.2. Email
PingCastle can contact if specified a SMTP server to send the reports by email. If the encryption is set, the program will encrypt the reports. Use –sendXmlTo <email> to send only the xml report, –sendHtmlTo <email> to send only the html report and –sendAllTo <email> to send both html and xml report. Email addresses are comma separated ones and the previous flags can be combined.

3.3. API

PingCastle can send the report (encrypted or not) using an API.

You can query a PingCastle API server or build a client or server from Swagger.

The description of the API in swagger format can be found here.

Build it

The report can be generated in the interactive mode by choosing “healthcheck” or just by pressing Enter. Indeed it is the default analysis mode.
It can be run using the command:

PingCastle.exe –hc-conso
Note: This report is generated automatically when the healthcheck is performed with the server “*”

The special file ad_gc_entitymap.xlsx is used to provide business input to PingCastle reports.

Run the program PingCastleReporting and enter “template” in the interactive mode. An empty ad_gc_entitymap.xlsx will be created. As an alternative, run the command:

PingCastleReporting.exe --gc-template

Run the program PingCastleReporting and enter in the interactive mode “conso”. As an alternative, run the command:

PingCastleReporting.exe --gc-overview

The program will load the file ad_gc_entitymap.xlsx in the current path and produce the Powerpoint file ad_gc_overview.pptx.

The template used to generate can be exported with the flag –export-pptx-template, modified, then loaded using the flag –use-pptx-template. Only slides with special notes will be altered

The report can be generated in the interactive mode by choosing “healthcheck” or just by pressing Enter. Indeed it is the default analysis mode.
It can be run using the command:

PingCastle.exe --graph --server mydomain.com

The report can be generated in the interactive mode by choosing “scanner” or just by pressing Enter. Then the list of available scanner is displayed.

As an alternative, the scanner can be run using the command:

PingCastle.exe --scanner <type> --server mydomain.c
  
 Scanners
There is 6 available scanners.

localadmin/ms17-010/replication/share/smb/startup
This module enumerates the local admin accounts on the workstations and servers of the domain.
