# IPList
List of bad and good IPs which was earlier used for [fumbled.lol](https://fumbled.lol).

Updates every hour

## Threat IPs
We use multiple threat ip sources from various aggrevated lists that. These should not include confirmed VPN/Tor IPs, but it may vary.

Blocklists used:
- Firehol
  - Firehol Level1
  - Firehol Level2
  - Firehol proxies
  - [Botscout 7d](https://iplists.firehol.org/?ipset=botscout_7d)
  - DShield 7d
  - Cleantalk 1d
- Netmountains Curated IP Blacklist
- ThreatFox IOC IPs
- IPSum Level 2
- Data-Shield IPv4
- Binarydefense Banlist
- Greensnow
- Cinsscore CI Badguys
- Emergingthreats Compromised IPs
- abuse.ch feodotracker aggressive
- StopForumSpam Toxic IPs
- AbuseIPDB 100% confidence 30d

## Good bots
Our implementation for good bots were that it could bypass firewall with light ratelimiting, and blocking POST, PUT, DELETE etc requests.

<details>
  <summary>HAProxy config</summary>
  
  ```haproxy
  acl is_read_only method GET HEAD
  acl is_good_bot src -f /etc/haproxy/ip_list/good_bots.lst
  http-request deny if is_good_bot !is_read_only
  http-request allow if is_good_bot
  ```
</details>

## Non-residential ip traffic
Our implementation for non-residential ip traffic was to block Tor and Datacenter for non-GET method requests. Threat IPs are instantly blocked. Generally allow read traffic especially if you are using Cloudflare and implementing correct cache rules.

VPN traffic should be allowed, though with some ratelimits for example to API routes.

<details>
  <summary>HAProxy config</summary>
  
  ```haproxy
  acl is_tor src -f /etc/haproxy/ip_list/tor_ips.lst
  acl is_vpn src -f /etc/haproxy/ip_list/vpn_ips.lst
  acl is_datacenter src -f /etc/haproxy/ip_list/dc_ips.lst
  acl is_malicious src -f /etc/haproxy/threat-intel/threats.lst

  http-request deny deny_status 403 if is_malicious !is_tor !is_vpn

  acl is_write_method method POST PUT PATCH DELETE
  http-request deny deny_status 403 if is_tor is_write_method
  http-request deny deny_status 403 if is_datacenter !is_vpn is_write_method
  ```
</details>

### Stripe whitelist
There is also Stripe's webhook IPs that should be whitelisted and must bypass WAF.

<details>
  <summary>HAProxy config</summary>

  Edit is_stripe_webhook_path ACL to your liking or remove it.
  
  ```haproxy
  acl is_stripe_webhook_ip src -f /etc/haproxy/ip_list/stripe_webhook_ips.lst
  acl has_stripe_signature hdr_cnt(Stripe-Signature) gt 0
  acl is_stripe_webhook_path path_beg /api/webhooks/stripe

  http-request deny if is_stripe_webhook_path !is_stripe_webhook_ip
  http-request deny if is_stripe_webhook_path !has_stripe_signature
  http-request allow if is_stripe_webhook_path
  ```
</details>