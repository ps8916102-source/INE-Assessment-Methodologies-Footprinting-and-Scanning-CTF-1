# Assessment Methodologies: Footprinting and Scanning CTF 1-walkthrough

> **Exercise:** INE Reconnaissance Skill Check  
> **Target:** `target.ine.local` (`http://target.ine.local`)  
> **Scope:** Authorized INE training environment only  
> **Safety:** Actual flag values and the recovered database password are redacted.

## 1. Lab objective

The lab asks for four flags. The clues point to HTTP response headers, `robots.txt`, anonymous FTP access, and a MySQL database listing.

## 2. Flag 1 — Inspecting the HTTP server header

### Command used

```bash
nmap -sV -sC -T4 -Pn target.ine.local
```

### Command breakdown

- `nmap` runs the network scanner.
- `-sV` probes open ports to identify service versions.
- `-sC` runs Nmap's default NSE scripts.
- `-T4` selects a faster timing template.
- `-Pn` skips host discovery and treats the target as online.
- `target.ine.local` is the lab hostname.

### Observed result

Nmap identified port `80/tcp` as HTTP and reported `Werkzeug/3.0.6 Python/3.10.12`. Its HTTP script output also showed an unusual `Server` header containing the flag. The actual value is omitted.

The scan also identified FTP on port 21 and MySQL on port 3306, which were relevant to later clues.

### Interpretation

The clue referred to inspecting HTTP response headers, not just the visible page. Headers can disclose implementation details and, in this lab, the unusual header carried the flag.

## 3. Flag 2 — Following `robots.txt` in Firefox

These were browser-based steps, not terminal commands:

1. Open `http://target.ine.local/robots.txt` in Firefox.
2. Inspect the directives. The file included:
   - `User-agent: googlebot` with `Disallow: /photos`
   - `User-agent: *` with `Disallow: /secret-info/` and `Disallow: /data/`
3. Open `http://target.ine.local/secret-info/` in Firefox.
4. The directory listing displayed `flag.txt`.

### Why it matters

`robots.txt` communicates crawl preferences to cooperative crawlers. Its `Disallow` entries do **not** protect the listed paths from direct access. Sensitive resources need proper authentication and authorization.

## 4. Flag 3 — Anonymous FTP access

The first Nmap scan reported `21/tcp` open, running `vsftpd 3.0.5`, with anonymous FTP login allowed. The listing included `creds.txt` and `flag.txt`.

The transcript also contains this follow-up command:

```bash
nmap -P 21 -sV --script=ftp-syst,ftp-anon target.ine.local
```

It produced the warning `setup_target: failed to determine route to 21 (0.0.0.21)`. Uppercase `-P` is not the port-selection option; lowercase `-p` is. A corrected version is:

```bash
nmap -p 21 -sV --script=ftp-syst,ftp-anon target.ine.local
```

The original command is preserved here as part of the learning history, not recommended syntax.

### FTP session used

```text
ftp target.ine.local
```

At the username prompt, `anonymous` was entered and accepted. At the FTP prompt, these commands were used:

```text
ls
get creds.txt
get flag.txt
exit
```

- `ftp target.ine.local` starts an FTP client connection to the lab host.
- `anonymous` requests an anonymous FTP login.
- `ls` lists files in the remote directory.
- `get creds.txt` downloads the credentials file to the local working directory.
- `get flag.txt` downloads the flag file.
- `exit` closes the FTP session.

### Local file inspection

After leaving FTP, `ls` showed that both files had been downloaded. They were inspected with:

```bash
cat flag.txt
cat creds.txt
```

`cat flag.txt` displayed Flag 3; its value is omitted. `cat creds.txt` revealed the username `db_admin` and a password, which is redacted.

### Interpretation

Anonymous FTP exposed both a flag file and database credentials. Publicly accessible files should not contain secrets, and anonymous access should be disabled unless explicitly required and carefully restricted.

## 5. Flag 4 — Connecting to MySQL and listing databases

### First connection attempt

```bash
mysql -h target.ine.local -u db_admin
```

This failed because no password was supplied. That error helped establish why the password prompt was needed.

### Successful connection

```bash
mysql -h target.ine.local -u db_admin -p
```

- `mysql` launches the database command-line client.
- `-h target.ine.local` specifies the database server hostname.
- `-u db_admin` specifies the database username.
- `-p` prompts for a password rather than placing it directly in the command.

After entering the recovered password interactively, the client connected successfully.

### Database enumeration

At the database prompt, the following SQL statement was used:

```sql
SHOW DATABASES;
```

It lists databases visible to the connected account. The output included a database name containing Flag 4, alongside standard system databases. The actual flag value is omitted.

The client was then closed with:

```sql
exit
```

The transcript does not show table queries, so this write-up does not claim that any tables were inspected.

## 6. Consolidated methodology

1. Scan the target to identify open services, versions, and useful script output.
2. Inspect HTTP response headers for unusual server information.
3. Open `robots.txt` in Firefox and follow the `/secret-info/` path.
4. Investigate FTP and verify whether anonymous login is allowed.
5. Retrieve and inspect the exposed files.
6. Use the recovered database credentials to connect to MySQL.
7. List databases and identify the database name containing the final flag.

## 7. Defensive lessons

- **Review response headers:** Remove unnecessary disclosures and never place secrets in headers.
- **Do not treat `robots.txt` as security:** Use server-side access controls for sensitive paths.
- **Restrict FTP access:** Disable anonymous login when it is not required and review remotely accessible files.
- **Protect credentials:** Do not store database passwords in publicly downloadable files. Rotate exposed credentials in a real incident.
- **Limit database privileges:** Use dedicated accounts with only the permissions required.
- **Audit exposed services:** Review whether each open port is necessary and apply appropriate network restrictions.

## 8. Publication checklist

- [ ] Actual flag values are omitted.
- [ ] The recovered database password is omitted.
- [ ] Screenshots and terminal output are checked for secrets and restricted lab details.
- [ ] The incorrect `-P 21` command is labeled as the original command, not recommended syntax.
- [ ] Claims are limited to what the transcript and screenshots support.
- [ ] INE's current publication rules for Skill Check solutions have been checked.

## 9. Final takeaway

This lab connected several reconnaissance findings: an unusual HTTP header, a path listed in `robots.txt`, anonymous FTP files, and credentials that granted database access. Follow evidence step by step, explain what each result demonstrates, and document the work accurately without exposing secrets.
