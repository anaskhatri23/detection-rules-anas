# Azure Entra Authentication Attempts from Abused Hosting Service Providers

---

## Metadata

- **Author:** Elastic
- **Description:** This hunting query gathers evidence of Azure Entra authentication attempts from hosting service providers that are often abused by adversaries. By identifying authentication attempts from these sources, security teams can detect potential unauthorized access or malicious activities within their Azure environment.

- **UUID:** `d27f1da8-eec6-11ef-983a-f661ea17fbce`
- **Integration:** [azure](https://docs.elastic.co/integrations/azure)
- **Language:** `[ES|QL]`
- **Source File:** [Azure Entra Authentication Attempts from Abused Hosting Service Providers](../queries/entra_authentication_attempts_from_abused_hosting_service_providers.toml)

## Query

```sql
FROM logs-azure.signinlogs-*

// query Azure Entra Sign-in logs
| WHERE @timestamp > now() - 14 day
| WHERE
    event.dataset in ("azure.signinlogs") and

    // filter for authentication events
    event.category == "authentication" and

    // filter for specific ASN organizations that are often abused
    source.as.organization.name in (
        "DigitalOcean", "Linode", "Vultr", "Hetzner", "OVH",
        "Contabo", "Leaseweb", "G-Core Labs", "Scaleway", "Kamatera",
        "Shinjiru", "M247", "Packet Host", "InterServer", "DataPacket",
        "Choopa", "Path Network", "DediPath", "Maxided", "Quasi Networks",
        "FlokiNET", "Njalla", "AbeloHost", "Inferno Solutions", "Hostinger",
        "Hostwinds", "1&1 IONOS", "DreamHost", "A2 Hosting", "Bluehost",
        "Namecheap Hosting", "FastComet", "InMotion Hosting", "SiteGround", "GreenGeeks",
        "Liquid Web", "Hurricane Electric", "Ubiquity Hosting", "Snel.com", "Coresite",
        "Eonix", "WebNX", "SharkTech", "Hivelocity", "Zenlayer",
        "RapidSeedbox", "SeFlow", "Nexeon Technologies", "NextArray", "Zare",
        "Clouvider", "TimeWeb", "YISP", "StackPath", "LuxVPS",
        "Terrahost", "IP Volume", "RackNerd", "ServerMania", "HostEurope",
        "HostHatch", "HostUS", "Cloudsigma", "QuadraNet", "CIV Host",
        "Swiftway", "King Servers", "BeeHost", "Webzilla", "Flokinet",
        "Alexhost", "DDoS-Guard", "StormWall", "Yokohama Networks", "DataGroup",
        "GSL Networks", "MyLoc", "Hostlife", "Reprise Hosting", "GTT Communications",
        "Telia", "Cogent Communications", "NForce", "Ecatel", "Novogara",
        "CyberBunker", "DarkFiber", "Exoscale", "QHoster", "ServDiscount",
        "Krypt", "Wowrack", "XLHost", "OVHCloud", "Privex",
        "GreencloudVPS", "RamNode", "BuyVM", "LiteServer", "Host1Plus",
        "EdgeUno", "CloudSouth", "IOFlood", "Hostry", "MivoCloud",
        "CloudCone", "SwiftNode", "Flaunt7", "Infinitie Networks", "ServerHub",
        "Verpex Hosting", "W3Space", "HostPapa", "Storm Internet", "WP Engine",
        "Kinsta", "Fly.io", "Edgecast", "RocketNode", "CloudAtCost",
        "Gullo's Hosting", "Serverion", "XHostFire", "Interserver", "DediServe",
        "HostRound", "VPSServer", "HostMantis", "RapidSwitch", "Tiggee LLC",
        "LogicWeb", "VPSCheap", "Versaweb", "SecureDragon", "ServerAstra",
        "HostNeverDie", "CloudSigma", "IONOS Cloud", "StackClash", "ProtonVPN",
        "NordVPN", "Mullvad VPN", "ExpressVPN", "Surfshark", "Private Internet Access",
        "TorGuard", "VyprVPN", "Windscribe VPN", "iVPN", "Perfect Privacy",
        "Astrill VPN", "TunnelBear", "CyberGhost VPN", "PureVPN", "Hotspot Shield",
        "StrongVPN", "F-Secure VPN", "IVPN", "ZoogVPN", "SwitchVPN",
        "AirVPN", "Buffered VPN", "SecureVPN", "RiseupVPN", "Betternet",
        "Trust.Zone VPN", "OneVPN", "VeePN", "Speedify VPN", "VPN Unlimited",
        "Anonine VPN", "X-VPN", "Hidemyass VPN", "ProXPN", "VPNArea",
        "AceVPN", "IPVanish", "Bitmask VPN", "BolehVPN", "AnonVPN",
        "Librem One VPN", "BlackVPN", "Cloudflare Warp VPN", "Torguard VPN", "VPN.ht"
    )

// aggregate authentication attempts by tenant, user principal name, authentication protocol, category, ASN organization name, and source address
| STATS asn_count = count(*) by
    azure.tenant_id,
    azure.signinlogs.properties.user_principal_name,
    azure.signinlogs.properties.authentication_protocol,
    azure.signinlogs.category,
    source.as.organization.name,
    source.address
```

## Notes

- Review `azure.signinlogs.properties.authentication_protocol` to determine the authentication method used.
- Device Code Flow authentication is particularly suspicious for non-kiosk, non-IoT devices.
- Analyze `source.as.organization.name` to identify if the authentication originated from a known hosting provider, VPN, or anonymization service you are not expecting
- Investigate `source.address` to determine if the IP address has been previously flagged for suspicious activity.
- Pivot on `azure.signinlogs.properties.user_principal_name` to check for additional authentication attempts from different IPs, ASNs, or unusual geolocations.
- Correlate findings with `azure.signinlogs.properties.authentication_processing_details` to check if legacy protocols, token replay, or bypass mechanisms were involved.
- Look for associated Conditional Access policy decisions in `azure.signinlogs.properties.applied_conditional_access_policies` to determine if the authentication attempt was blocked or allowed.
- Check `azure.signinlogs.properties.device_detail.browser` to see if the user agent is consistent with expected authentication patterns.
- If authentication was successful, investigate any subsequent suspicious activity linked to the same user session.

## MITRE ATT&CK Techniques

- [T1078.004](https://attack.mitre.org/techniques/T1078/004)

## References

- https://www.volexity.com/blog/2025/02/13/multiple-russian-threat-actors-targeting-microsoft-device-code-authentication/
- https://www.blackhillsinfosec.com/dynamic-device-code-phishing/

## License

- `Elastic License v2`
