# Hands-on INE Assessment Methodologies: Footprinting and Scanning CTF 1 walkthrough

> **Focus:** HTTP response headers, `robots.txt`, anonymous FTP access, and MySQL reconnaissance  
> **Environment:** Authorized INE Skill Check lab  
> **Target:** `http://target.ine.local`  
> **Publication safety:** Actual flag values and the recovered database password are omitted.

## Overview

This repository documents an authorized INE reconnaissance Skill Check. The objective was to inspect the target web server, follow clues exposed by its responses and crawler directives, investigate anonymous FTP access, and use recovered credentials to inspect MySQL databases.

## Objectives and findings

| Objective | Investigation | High-level finding |
|---|---|---|
| Flag 1 | Inspect HTTP response headers | An unusual `Server` header contained the flag. |
| Flag 2 | Inspect `robots.txt` in Firefox and follow a disallowed path | `/secret-info/` displayed a directory listing containing `flag.txt`. |
| Flag 3 | Enumerate FTP and log in anonymously | Anonymous FTP exposed `creds.txt` and `flag.txt`. |
| Flag 4 | Use recovered credentials to connect to MySQL | The database listing included a database name containing the flag. |

## Repository contents

- [`lab-notes.md`](lab-notes.md) — detailed methodology, command explanations, observations, troubleshooting, and defensive lessons.

## Tools

- **Nmap** — service/version detection and relevant scripts.
- **Firefox** — inspection of `robots.txt` and the discovered path.
- **FTP client** — anonymous login and file retrieval.
- **MySQL client** — database connection and database enumeration.

The lab transcript records Nmap, FTP, Firefox, and MySQL activity. Follow the active INE lab rules when reproducing the exercise.

## Key lessons

- HTTP headers can disclose unexpected information.
- `robots.txt` is crawler guidance, not access control.
- Anonymous FTP can expose files and credentials.
- Credentials found in one service may provide access to another.
- Public notes should explain the evidence without publishing secrets.

## Responsible publication

This is a record of activity in an authorized training environment. Public versions omit actual flag values and the recovered password. Check INE's current publication rules before publishing detailed Skill Check solutions, and never test these techniques against systems without explicit authorization.
