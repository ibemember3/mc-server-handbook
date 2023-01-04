# Technical handbook for Minecraft Java servers

This document is not a guide/tutorial on making a Minecraft server. It is a personal handbook (that I'm choosing to publish publicly for some reason) containing technical information on setting up and maintaining a server. This handbook does not cover the community aspect of running a server.

___

## Server Hardware

### CPU

Whilst many optimisations are in various Minecraft server software to utilise more cores/threads, the main Minecraft server-tick loop only runs on one thread; this means that whilst pre-generation of worlds using forks such as PaperMC can benefit significantly from more cores, the day-to-day performance will not dramatically improve after two cores.

With that in mind, choosing a CPU with a very high single-thread rating is essential. The **Ryzen 9 5950X** serves as the golden standard CPU for a Minecraft server with a single thread rating of 3466.

### RAM

Minecraft servers greatly benefit from more RAM but quickly reach a point of diminishing return when increasing specs. Whilst better RAM specs can help other services running on the same box, Minecraft servers (especially when running PaperMC and initiated with Aikar's flags) tend to show degraded performance after around the **10-14GB** mark.

A **DDR4-3200** RAM set is the golden standard, with DDR4-2400 serving as a close 2nd.

### Storage

Minecraft servers' performance increases dramatically with higher random read/write speeds as players load in and modify random chunks. An **NVMe M.2 SSD** serving as the drive hosting the Minecraft server paired with two HDDs in a RAID 1 configuration for scheduled on-site backups is a good setup for many Minecraft servers.

### Network

Minecraft servers will run fine on almost any network speed but **gigabit internet** is the standard. Using a server in a proper data centre environment will yield lower pings for players because the data centre will likely have peered with significant ISPs and networks.

## Server Software

### "Vanilla"-like Servers

For many years, Spigot has been the most stable fork of Bukkit that provides plugin functionality. However, with each new Minecraft version, performance has steadily dropped. Many forks of Spigot have popped up recently, with **PaperMC** serving as the primary fork to include *many* stable performance improvements at the cost of some pure vanilla functionality.

Some servers may require more built-in customisation without compromising Paper's excellent performance. Therefore **Purpur** (a PaperMC fork) can be a drop-in replacement.

### Modded Servers

**Forge** provides the most mods for servers and which is the most familiar to players. However, Forge is extremely demanding Minecraft server software and requires expertise to run for large servers.

**SpongeForge** combines Forge's excellent modding capabilities (and is compatible with Forge mods) with its plugin ecosystem. It intends to bridge the gap between the power of Forge's mods and the versatility of plugins.

**Fabric** is an alternative to Forge with a different modding API. Fabric is lightweight and offers excellent performance and feature parity with vanilla Minecraft (great for technical Minecraft players creating large, complex contraptions). However, Fabric does not provide as many mods as Forge.

### Hybrid Servers

Many Minecraft server software exists that (claim to) provide a bridge between Forge mods and Spigot (and its forks) plugins. These are often outdated, broken, and extremely slow. Many Minecraft server hosting companies will even *refuse* to support these types of servers. **Stay away from them.**

## Security

### DDoS Protection

Minecraft servers that become large enough will attract people who seek to ruin the experience. It is, therefore, vital for a server to prevent DDoS.

#### Ready Solutions

Many solutions have been created (and often for a heavy price) to provide protection. Below is a quick summary of various providers and their costs in USD per TB.

+ TCPShield: Uses OVHCloud's DDoS prevention + custom rules. Free up to 1TB of bandwidth a month. $25 USD for 2.5TB a month. $100/$250 USD a month for unlimited bandwidth.
+ Cloudflare Spectrum: Uses Cloudflare's **excellent** systems (they secure 19.4% of all websites). **EXPENSIVE** with $1000 USD for 1TB.
+ MCShield: Uses Cloudflare Spectrum. $10 USD a month. Not suitable for servers outside of a friend group, as you cannot use your own domain.
+ Lectron: Uses Cloudflare Spectrum and Path.net. $33.33 USD for 1TB.
+ CosmicGuard: Custom solution. Weird, opaque pricing - $0.03 average per player daily (???)
+ PlayIt.gg: Custom solution - bit rudimentary. Free. If you want your dedicated IP and wish to avoid SRV records, $10 USD a month ($100 USD a year).

[Further Reading](https://github.com/BlockhostOfficial/ddos-prot-mc-servers "Further Reading")

#### "Homemade" Solutions

Utilising the DDoS protection of many VPS providers, it is possible to use a Minecraft proxy to proxy traffic from the DDoS-protected VPS to your Minecraft server. Popular VPS providers include **Hetzner**, **OVHCloud**, and **BuyVM**.

If proxying, it is crucial to use the firewall on the Minecraft server box to **block incoming connections from everyone except the proxy VPS**.

### SSH and FTP/SFTP

Use **SSH keys** to SSH and transfer files. Passwords are bad, but you won't die if you use them.

### Server Plugins

**Vet all plugins** before installing them. Even popular plugins by well-known authors have been known to contain malware due to their accounts being hacked.

### Server Panels

While using internal, back-end game panels can make your less technically-savvy game admins' lives more manageable with a friendly GUI, they are often exposed incorrectly to the internet, contain default login credentials, and include security holes. Setup **2FA**, **remove default accounts with poor passwords** and **research** to ensure the panel you're installing is safe.

### Server Software

Keep **up-to-date** on server news and updates on your server software. The infamous Log4j exploit in 2021 was one of the most dangerous, allowing unprecedented remote code execution on nearly all Minecraft clients and servers. Many Minecraft servers are still vulnerable today as a result of them not being updated. **Update your servers.**

### Indirect Vulnerabilities

Even if your server is secured very well, it is only as secure as you. If you use DiscordSRV and link a #console channel in your Discord server, a hacker can target your Discord account instead to gain access to your Minecraft server. Use **2FA** and **strong passwords** on all your other accounts.

This also applies to people you hire to help with your server.

## Aikar's Flags

Java Virtual Machine (JVM) Garbage Collection can negatively impact Minecraft servers. Memory can often get cleaned up at incorrect times, improperly utilised for Minecraft servers, and other issues lead to a degraded Minecraft server.

Aikar, a developer within the Minecraft server scene, has extensively researched JVM factors affecting the performance of Minecraft servers and has designed two sets of flags to launch a server with to maximise performance.

1-12GB RAM:

`-Xms**RAM**G -Xmx**RAM**G -XX:+UseG1GC -XX:+ParallelRefProcEnabled -XX:MaxGCPauseMillis=200 -XX:+UnlockExperimentalVMOptions -XX:+DisableExplicitGC -XX:+AlwaysPreTouch -XX:G1NewSizePercent=30 -XX:G1MaxNewSizePercent=40 -XX:G1HeapRegionSize=8M -XX:G1ReservePercent=20 -XX:G1HeapWastePercent=5 -XX:G1MixedGCCountTarget=4 -XX:InitiatingHeapOccupancyPercent=15 -XX:G1MixedGCLiveThresholdPercent=90 -XX:G1RSetUpdatingPauseTimePercent=5 -XX:SurvivorRatio=32 -XX:+PerfDisableSharedMem -XX:MaxTenuringThreshold=1 -Dusing.aikars.flags=https://mcflags.emc.gs -Daikars.new.flags=true`

12GB+ RAM:

`-Xms**RAM**G -Xmx**RAM**G -XX:+UseG1GC -XX:+ParallelRefProcEnabled -XX:MaxGCPauseMillis=200 -XX:+UnlockExperimentalVMOptions -XX:+DisableExplicitGC -XX:+AlwaysPreTouch -XX:G1NewSizePercent=40 -XX:G1MaxNewSizePercent=50 -XX:G1HeapRegionSize=16M -XX:G1ReservePercent=15 -XX:G1HeapWastePercent=5 -XX:G1MixedGCCountTarget=4 -XX:InitiatingHeapOccupancyPercent=20 -XX:G1MixedGCLiveThresholdPercent=90 -XX:G1RSetUpdatingPauseTimePercent=5 -XX:SurvivorRatio=32 -XX:+PerfDisableSharedMem -XX:MaxTenuringThreshold=1 -Dusing.aikars.flags=https://mcflags.emc.gs -Daikars.new.flags=true`

Replace `**RAM**` with the amount of RAM you wish to allocate to your Minecraft server.

[Further Reading](https://aikar.co/2018/07/02/tuning-the-jvm-g1gc-garbage-collector-flags-for-minecraft/ "Further Reading")

## Dynamic Maps

Some, often small survival, Minecraft servers offer a web map that allows people to view and sometimes chat with the Minecraft server. However, setting one up securely and properly requires more technical configuration.

Many web maps do not provide HTTPS, which therefore requires a reverse proxy such as NGINX to provide the security needed (and other benefits). Directly exposing a port to the internet can be a dangerous gamble (and will not work if you're doing a "homemade" DDoS protection method described earlier), so using a tunnel such as Cloudflare Tunnel can be an excellent, safe way to expose a service.

Web maps also use a lot of storage, so it is a good idea to lower default rendering resolutions in their configuration files. A database is also good if the web map supports it.
