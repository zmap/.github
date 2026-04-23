# Welcome to The ZMap Project

The ZMap Project is a collection of open source tools for performing large-scale studies of the hosts and services that make up the public Internet.

## Getting Started

New here? Start with one of these guides:

- **[Getting Started with ZMap](https://github.com/zmap/zmap/wiki/Getting-Started-Guide)** — a step-by-step walkthrough of basic Internet-wide scanning with ZMap.
- **[Getting Started with ZMap + ZGrab2](../wiki/getting-started-with-zmap-and-zgrab2.md)** — a step-by-step introduction to using ZMap and ZGrab2 together with helpful CLI utilities and examples

Before scanning, please review our **[Scanning Best Practices](https://github.com/zmap/zmap/wiki/Scanning-Best-Practices)**.

---

## Core Tools

### [ZMap](https://github.com/zmap/zmap)

A fast single-packet network scanner optimized for Internet-wide surveys. On a gigabit connection, ZMap can sweep the entire public IPv4 address space on a single port in under 45 minutes with a 1 Gb/s connection.

### [ZGrab2](https://github.com/zmap/zgrab2)

A modular, stateful application-layer scanner designed to complement ZMap. Where ZMap identifies responsive hosts at the network layer, ZGrab2 conducts protocol handshakes and collects banners across HTTP, HTTPS, TLS, SSH, FTP, SMTP, and many more.

### [ZDNS](https://github.com/zmap/zdns)

A high-performance DNS lookup tool with its own recursive resolver. ZDNS supports a wide range of record types (A, AAAA, MX, TXT, CAA, DMARC, and more), making it well-suited for large-scale DNS measurement studies.

### [ZAnnotate](https://github.com/zmap/zannotate)

A utility for enriching Internet datasets with contextual metadata. ZAnnotate can annotate IP addresses with geolocation data (MaxMind GeoIP2), autonomous system information, routing data from MRT files, intelligence services like Censys and GreyNoise, and more.

---

## Community & Support

- 🐛 **Found a bug?** Open an Issue in the relevant repository.

---

## Contributing

We'd welcome contributions for both documentation or development. For larger changes or features, please confirm with an Issue on the relevant repository that the idea is something we'd want to incorporate so there's no misalligned expectations or disappointment if we're not able to merge.
