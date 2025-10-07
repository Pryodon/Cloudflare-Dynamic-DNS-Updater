# Cloudflare Dynamic DNS Update PHP script

If you have your own domain name, you can use Cloudflare's free services to get free dynamic IP updates using this script! Why pay DynDNS for this when this script does the same thing on your own domain name for free?! Don't have a domain name? Cloudflare offers wholesale priced domain names at their cost. I have all of my domain names registered with Cloudflare. .com and .net domains are like $11 a year and included is free DNS hosting with free one-click DNSSEC setup! Cloudflare even offers free DNS hosting for your domain names registered at another registrar! 

This script updates your current dynamic IP address in your DNS records which you have hosted at Cloudflare. If you want, you can also proxy your website through Cloudflare's proxy to hide your home IP address. 


**Cloudflare Dynamic DNS Update PHP script**  *


**WHAT THIS DOES**
- This PHP script updates your domain name's DNS to point to your home IP address.

- Example: If your home IP address is: 8.8.8.8
     In the DNS, host.example.com will point to 8.8.8.8

- There are for pay services that do what this script does, but with this script,
  it is totally free! All you need is this script and your own domain name with
  the DNS hosted for free at Cloudflare. FYI, Cloudflare sells .com and .net domain
  names for super cheap!! About $11 per year. Some domain name sellers offer a name
  for like $3 a year for the first year, but then they jack up the price to like $30
  per year.

**HOW IT WORKS**
- Checks your current public home IP using Cloudflare's 1.1.1.1 trace endpoint.
- Looks up the A record in Cloudflare DNS for host.example.com.
- Only updates the record if your IP changed (avoids unnecessary API calls).

**RECOMMENDED EDITOR**
Use a code editor to edit this script that respects Unix line endings
(e.g., Notepad++ on Windows).
Download: https://notepad-plus-plus.org/

**TYPICAL HOST**
Runs great on a small always-on Linux box (e.g., Raspberry Pi 5).
https://www.raspberrypi.com/products/raspberry-pi-5/

**SETUP (Debian/Ubuntu examples)**

**1) Install prerequisites**
   From your Linux shell:
   $ sudo apt update
   $ sudo apt install php php-curl curl

**2) Create a Cloudflare API token**
   Go to https://dash.cloudflare.com/
   - Cloudflare Dashboard -> My Profile -> API Tokens -> Create Token
   - Use the "Edit zone DNS" template.
   - Scope it to only your target zone (domain) for safety.
   Required permission: Zone -> DNS -> Edit

**3) Find your Zone ID**
   - Cloudflare Dashboard -> Select your domain -> Overview (right side) -> Zone ID
   (Optional via API, replace token below)
   Note: The long number after "Bearer" below is where you put your API token.
   Run this command to get your Zone ID:

   curl -X GET "https://api.cloudflare.com/client/v4/zones" -H "Authorization: Bearer 0000000000000000000000000000000000000000" -H "Content-Type: application/json"

   # Look in the JSON for: "result":[{"id":"00000000000000000000000000000000", ...
                                               -- This is your Zone ID --

**4) Configure this script (see CONFIG section below)**
   - $api_token: Cloudflare API token with Zone:DNS:Edit permission
   - $zone_id:   Cloudflare Zone ID (NOT the domain name)
   - $record_name: host to update (e.g., host.example.com or example.com)

   Note: The hostname to update must already exist in your DNS.
         If you don't have a DNS A record, create one and for the IP make it
         be 0.0.0.0 ... this script will change the IP when it runs.

**5) Run this script first to make sure it updates your DNS correctly.**
   $ php cloudflare-dynamic-DNS-update.php

   After you run this script, look up your domain name you configured to see if it is correct.
   $ host host.example.com

   You can see your public IP on this web page:
   https://1.1.1.1/cdn-cgi/trace

**6) Schedule with cron**
   This crontab entry below runs this script every 5 minutes.
   If your home IP has changed, it updates your DNS.

   Edit your crontab:
   $ crontab -e
   Add this line below (adjust PHP and script paths):
   Note: Only copy/paste the part between the quotes, but don't include the quotes.
*/
//   "*/5 * * * * /usr/bin/php /path/to/cloudflare-dynamic-DNS-update.php -q >/dev/null 2>&1"
/*
   Notes:
   - The -q flag suppresses normal output; errors still log to the history file.
   - Ensure the user running cron can write to the log file paths set below.

**OPTIONAL LOGIN DISPLAY**
- $login_display_file prints "current IP + last-change time".
  This displays your current home IP address when you login to your shell.
  Add to the end of ~/.bashrc (or ~/.profile):
    if [ -f /home/user/log/ip-change-display.txt ]; then
        cat /home/user/log/ip-change-display.txt
    fi

**LOGGING**
- $history_log_file appends a timestamp each time the IP changes (no IP values).
- Errors are appended with an "ERROR:" prefix and timestamp.


**// ==== CONFIG ====**
$api_token    = '0000000000000000000000000000000000000000'; // Cloudflare API token (Zone:DNS:Edit)
$zone_id      = '00000000000000000000000000000000';         // Cloudflare Zone ID (not the domain name)
$record_name  = 'host.example.com';                         // Hostname/subdomain to update; use example.com for root A record

// Paths to your log files - change to wherever you want them
$login_display_file = '/home/user/log/ip-change-display.txt'; // For login display (current IP + last-change time)
$history_log_file   = '/home/user/log/ip-change-history.log'; // For history (timestamps when IP changed)

**// ==== END OF CONFIG ====**
