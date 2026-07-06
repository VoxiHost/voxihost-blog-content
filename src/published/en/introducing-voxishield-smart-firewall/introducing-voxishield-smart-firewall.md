---
image: /assets/images/blog/en/introducing-voxishield-smart-firewall/og-image.png
title: 'VoxiShield Smart Firewall Dashboard Controls Released'
description: Manage your VPS network security with VoxiShield! Configure inbound rules, rate limiting, egress filters, and GeoIP blocks directly from the dashboard.
date: '2026-07-10'
translationKey: introducing-voxishield-smart-firewall
locale: en
category: Updates
tags:
  - voxishield
  - firewall
  - ddos-protection
  - vps-security
  - announcement
status: published
author:
  name: VoxiHost Team
  link: https://voxihost.pl/
contributors:
  - danielmarszalkowski
faq:
  - question: "Is VoxiShield included with my service?"
    answer: "Yes! VoxiShield is included free with all VoxiHost services. Every virtual server comes with full edge filtering and custom firewall controls enabled from day one at no additional cost. You don't pay extra for DDoS protection, traffic scrubbing, or rule configuration - it's built into every plan."
  - question: "How does the two-layer protection work?"
    answer: "Traffic is analyzed and filtered progressively through two distinct defense layers:<br><br><b>Layer 1: Edge Scrubbing</b> - Our global edge network with 4+ Tbit/s capacity absorbs massive volumetric floods (like UDP/SYN floods) before they can reach our network core.<br><br><b>Layer 2: Core Firewall</b> - Cleaned traffic reaching our hypervisors is inspected by an intelligent core firewall. This is where your custom rules, port access, GeoIP filters, and rate limits are enforced with sub-second propagation."
  - question: "What types of attacks does VoxiShield protect against?"
    answer: "VoxiShield protects against a wide range of threat vectors: volumetric UDP/TCP floods, NTP/DNS amplifications, invalid protocol payloads (e.g. game query exploits for Minecraft, Rust, and FiveM), application layer (L7) HTTP floods, brute-force attempts, and port scans. Our edge templates automatically adapt to game and web traffic, dropping malicious packets while letting real users connect."
  - question: "Can I configure firewall rules myself?"
    answer: "Yes! Every customer gets full management access to the firewall via the VoxiHost dashboard. You can create custom port rules, restrict protocols, set packet rate limits, block specific countries using our GeoIP filter, toggle anonymous VPN/Tor network blocks, and inspect blocked connections in real-time via the live analyzer feed."
  - question: "What should I do during an active attack?"
    answer: "In most cases, you don't need to do anything at all. VoxiShield mitigates attacks automatically within seconds. If you notice any service degradation, you can check the live firewall log in your dashboard or contact our support team. We can immediately write custom Layer 2 rules or adjust edge filtering patterns for your specific application port."
  - question: "Does VoxiShield protection increase network latency?"
    answer: "No. Unlike traditional tunneling protections that route traffic through distant cleaning centers, our Layer 1 edge network is strategically located near main transit nodes, and Layer 2 filtering runs at the hypervisor level. This means security filtering happens inline without introducing routing loops or increasing latency, keeping game tickrates and api response times low."
---

At **<span class="text-white">Voxi</span><span class="text-amber-300">Host</span>**, network security has always been a core foundation, meaning every VPS we host is protected automatically from day one by our two-layer VoxiShield protection. Today, we are taking a massive leap forward by putting that power directly into your hands.

We are thrilled to announce the official release of the **VoxiShield DDoS Protection & Smart Firewall** management panel inside the VoxiHost dashboard! You are no longer restricted to default background filtering; you can now customize, monitor, and define your network security policies in real-time.

{% image "/assets/images/blog/en/introducing-voxishield-smart-firewall/dashboard-overview.png", "VoxiShield Firewall Overview Dashboard in the VoxiHost dashboard", "(max-width: 768px) 100vw, 800px" %}

---

## The Two-Layer Mitigation Pipeline

Traditional firewalls operate directly on the server operating system, consuming precious CPU and RAM resources during a network attack. VoxiShield filters malicious traffic in a progressive, off-loaded pipeline before it ever reaches your virtual machine.

### Layer 01: PletX Edge (Volumetric Filtering)
All incoming global traffic first passes through the **PletX Edge scrubbing network**. This layer is unmanaged and operates 100% automatically. It acts as a massive shield, soaking up volumetric DDoS floods (like DNS/NTP amplification or UDP floods) up to 4+ Tbit/s so that your server's bandwidth remains completely untouched.

### Layer 02: Core Firewall (Dashboard Managed)
Once volumetric threats are neutralized, traffic reaches our transparent **Core Firewall**. This is the layer you control. Directly from the <span class="text-white">Voxi</span><span class="text-amber-300">Host</span> dashboard, you can define custom traffic access rules, manage ports, configure rate limits, and block malicious traffic with sub-second propagation.

---

## Powerful Firewall Features at Your Fingertips

We designed VoxiShield to replace complex `iptables` and `nftables` commands with an intuitive, visual control center. You do not need to be a network administrator to secure your VPS.

### 1. Inbound Rules & Rate Limiting
Control exactly who and what can connect to your applications. With our Inbound Rules panel, you can:
*   Open specific ports or custom port ranges.
*   Restrict protocols to TCP, UDP, or both.
*   Define custom actions (Allow / Drop) for each rule.
*   Apply **precise packet-per-second (PPS) limits** to protect game servers and applications from connection floods.

{% image "/assets/images/blog/en/introducing-voxishield-smart-firewall/inbound-rules.png", "Configuring VoxiShield Inbound Rules and PPS limits in the VoxiHost dashboard", "(max-width: 768px) 100vw, 800px" %}

### 2. Egress Security (Outbound Control)
Keep your server's IP address clean and reputable. Outbound traffic control allows you to monitor and limit egress data, preventing a compromised application from scanning the internet or joining botnets.
*   **Default Port 25 (SMTP) Block** to prevent your VPS from sending spam emails.
*   Outbound custom port whitelist controls to restrict egress traffic to verified services.

{% image "/assets/images/blog/en/introducing-voxishield-smart-firewall/outbound-rules.png", "Configuring VoxiShield Outbound Egress security filters in the VoxiHost dashboard", "(max-width: 768px) 100vw, 800px" %}

### 3. Block Analyzer
Diagnosing network issues is now effortless. If a legitimate user or external service is having trouble connecting, you can instantly query their IP address in the **Block Analyzer**.
*   Verify if an IP is flagged by Global Inbound filters (such as GeoIP or ASN blocks).
*   Check if the IP is temporarily rate-limited due to exceeding your PPS rules.

{% image "/assets/images/blog/en/introducing-voxishield-smart-firewall/block-analyzer.png", "Querying blocked IP addresses in the VoxiShield Block Analyzer tool", "(max-width: 768px) 100vw, 800px" %}

### 4. Global Inbound Filters
Block whole vectors of malicious traffic before they even reach your server's network card:
*   **GeoIP Country Filter**: Allow or block traffic from specific countries.
*   **ASN Network Filter**: Block entire hosting providers, VPN networks, or ISPs.
*   **Threat Shield Intelligence**: Automatically block known malicious botnets, scanners, and exploit sources updated daily.

{% image "/assets/images/blog/en/introducing-voxishield-smart-firewall/global-filters.png", "Managing GeoIP and ASN Global Inbound filters in the VoxiHost dashboard", "(max-width: 768px) 100vw, 800px" %}

---

## Dedicated Game & Protocol Protection

Different services require different filtering rules. A website running on HTTPS needs web-specific scrubbing, while a Minecraft or Counter-Strike server requires low-latency UDP filtering. 

VoxiShield includes dedicated, automatically deployed scrubbing templates tailored for the most popular games and protocols. These templates adapt in real-time to mitigate zero-day exploits and keep your latencies stable.

---

## Special Launch Promotion: -35% OFF

To celebrate the release of the VoxiShield management interface, we are offering an exclusive discount on all our VPS hosting services. Protect your next project with built-in enterprise security at an unbeatable price!

Use promo code **<span class="text-amber-300 font-bold text-xl uppercase tracking-wider">VOXISHIELD</span>** at checkout to receive **35% OFF** your monthly billing cycle. 

> **Promo Details:** This code is valid for all monthly plans until **July 10, 2026**.

---

## Next Steps

With VoxiShield, enterprise-grade security is a standard, not a luxury. No surprise bills, no hidden bandwidth limits; only raw, high-performance protection that grows with your needs.

Ready to secure your infrastructure? Deploy a new [Budget VPS](/budget-vps/) or high-performance [Premium VPS](/premium-vps/) today, and manage your network policies through the [VoxiShield Smart Firewall](/shield/) interface inside the VoxiHost dashboard right away!
