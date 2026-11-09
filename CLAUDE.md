# CLAUDE.md

Guidance for Claude Code when working in this repository.

## What this is

`tvblocklists`: DNS blocklists of smart TV tracking, telemetry, ACR (automatic
content recognition) and ad domains, used with **pfBlockerNG DNSBL** on pfSense.
Remote: `https://github.com/voltpods/tvblocklists.git` (branch `main`).

- `smarttv-blocklist.txt`: smart TV list, **pfBlockerNG** format (rules below)
- `tuya-blocklist.txt`: Tuya / Smart Life IoT cloud, **AdGuard Home** format
- `README.md`: short public description

## tuya-blocklist.txt (AdGuard format; different from the TV list)

- Rules are `||domain^`, which also blocks all subdomains. Comments start with `!`.
- Keep the `! Title / Description / Homepage / Last modified / Expires` header,
  and update `Last modified` whenever you change the file.
- List only Tuya-owned **base** domains. Document the known subdomains
  (a1/m1/mq.gw/openapi/mqe/...) in comments instead of adding rules for them.
- Never block shared cloud domains (aliyuncs.com, amazonaws.com, myqcloud.com).
  Only add exact hostnames for those, taken from a real query log.
- Domains that aren't confirmed as Tuya-owned go in the commented-out
  UNVERIFIED section.
- Validate (should print nothing):
  `grep -v '^!' tuya-blocklist.txt | grep -v '^\s*$' | grep -Ev '^\|\|[a-z0-9.-]+\^$'`

The sections below describe the TV list.

## File format (keep it pfBlockerNG-compatible)

- One bare domain per line: no `0.0.0.0` prefix, no `http://`, no wildcards,
  no trailing dots, no inline comments after a domain.
- Lowercase only.
- `#` starts a comment line. Blank lines are fine.
- pfBlockerNG blocks subdomains when wildcard/TLD blocking is on, so don't add
  `a.example.com` if `example.com` is already listed.
- Plain LF line endings in the repo. Git on Windows warns about LF→CRLF, which
  is harmless.

## Structure

The header comment block (lines 1–8) sits at the top, followed by one section
per vendor in this order:

1. SAMSUNG (Tizen)
2. LG (webOS)
3. SONY / Philips / TP Vision
4. VIZIO
5. ROKU
6. TCL / Hisense / Vidaa / Toshiba
7. AMAZON FIRE TV
8. GOOGLE TV / ANDROID TV
9. ACR / Ad-tech used across many TV brands
10. HbbTV (EU broadcast red-button tracking)

Section header style:

```
# ===================== VENDOR NAME =====================
```

Add a new vendor as a new section above the cross-brand ACR/Ad-tech section.
A domain used by several brands goes in the ACR/Ad-tech section.

### "MAY BREAK" entries

Domains that can break firmware updates, app stores, or streaming apps sit at
the end of their vendor section, **commented out**, under a line like:

```
# MAY BREAK (what it breaks):
# domain.example
```

New domains that are risky like this go in as commented-out MAY BREAK entries,
never as active ones. Only uncomment one if the user explicitly asks.

## How to update the list

1. **Research / collect** the new domains. Prefer sources that observed real
   TV traffic (Pi-hole/pfBlockerNG community lists, privacy research papers,
   packet captures the user provides). Note where each domain came from in
   the commit message, not in the file.
2. **Classify** each domain as pure tracking/ads/ACR (active entry) or as
   something that also serves updates, logins, app stores, or streaming
   (commented-out MAY BREAK entry).
3. **Insert** it into the correct vendor section, keeping the existing
   grouping. Don't reorder or reformat the existing entries.
4. **Update** the `# Generated: YYYY-MM-DD` date in the header to today.
5. **Validate** before committing (run from the repo root in the Bash tool):

   ```bash
   # duplicates (should print nothing)
   grep -v '^\s*#' smarttv-blocklist.txt | grep -v '^\s*$' | sort | uniq -d
   # malformed lines: anything that isn't a bare lowercase domain (should print nothing)
   grep -v '^\s*#' smarttv-blocklist.txt | grep -v '^\s*$' | grep -Ev '^[a-z0-9]([a-z0-9-]*[a-z0-9])?(\.[a-z0-9]([a-z0-9-]*[a-z0-9])?)+$'
   # active domain count (mention it in the commit message)
   grep -v '^\s*#' smarttv-blocklist.txt | grep -vc '^\s*$'
   ```

   Also check for redundant subdomains: an entry whose parent domain is
   already listed.
6. **Commit and push** only when the user asks:

   ```bash
   git add smarttv-blocklist.txt
   git commit -m "Add <vendor> telemetry domains (N active)"
   git push
   ```

## Removing / unblocking

If the user reports that something broke (an app won't load, updates fail),
**comment out** the domain and move it under that section's MAY BREAK block
with a note on what it broke. Don't delete it. That keeps a record of why it
isn't active.

## Notes

- Environment: Windows Server, PowerShell plus a Bash tool. Git identity for
  this repo is set locally (`voltpods`).
- `README.mdgit` is a leftover paste of GitHub's setup commands. It's
  untracked and can be deleted.
