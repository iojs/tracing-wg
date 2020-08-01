# Diagnostics Working Group

[![Join us on nodeslackers.com (channel #diagnostics-wg)](https://img.shields.io/badge/Join%20us%20on-nodeslackers.com-blue)](https://node-js.slack.com/archives/C016U31J15M)

The goal of this WG is to ensure Node provides a set of comprehensive, documented, extensible diagnostic protocols, formats, and APIs to enable tool vendors to provide reliable diagnostic tools for Node.

Work is divided into several domains:
- [Tracing](./tracing)
- [Profiling](./profiling)
- [Heap and Memory Analysis](./heap-memory)
- [Step Debugging](./debugging)

Background, reference documentation, samples, and discussion for each domain is contained within its folder.

Work needed includes:
- Collect, understand, and document existing diagnostic capabilities and entry-points throughout Node, V8, and other components.
- Collect and document projects and products providing diagnostics for Node with brief description of their technical architecture and sponsoring organizations.
- Identify opportunities and gaps, then propose and implement solutions.

### Current Initiatives

| Initiative           | Champion                                         | Stakeholders                             | Links
|----------------------|--------------------------------------------------|------------------------------------------|-------------------------------------------------
| Diagnostic Channel   | [@qard](https://github.com/qard)                 |                                          | https://github.com/nodejs/diagnostics/issues/180
| Async Hooks          | [@qard](https://github.com/qard)                 |                                          | https://github.com/nodejs/diagnostics/issues/124
| Async Context        | [@mike-kaufman](https://github.com/mike-kaufman) | [@kjin](https://github.com/kjin)         | https://github.com/nodejs/diagnostics/issues/107
| Support tiers        | [@mhdawson](https://github.com/mhdawson)         |                                          | https://github.com/nodejs/diagnostics/issues/157
| CPU Profiling        | [@mmarchini](https://github.com/mmarchini)       |                                          | https://github.com/nodejs/diagnostics/issues/148
| Post-mortem WG merge | [@mmarchini](https://github.com/mmarchini)       |                                          | https://github.com/nodejs/post-mortem/issues/48
| Trace Events         | [@jasnell](https://github.com/jasnell)           | [@ofrobots](https://github.com/ofrobots) | https://github.com/nodejs/diagnostics/issues/84
| CLS                  | [@watson](https://github.com/watson)             |                                          | https://github.com/nodejs/node/pull/26540

#### Need volunteers for

| Initiative            | Champion      | Links                                            |
|-----------------------|---------------|--------------------------------------------------|
| Performance Profiles  |               | https://github.com/nodejs/diagnostics/issues/161 |
| Time-travel debugging |               | https://github.com/nodejs/diagnostics/issues/164 |
| Platform neutrality   |               |                                                  |

### Logistics

#### Semi-monthly Meetings

Meetings of the working group typically occur every other Wednesday. A few days before each
meeting, an [issue](https://github.com/nodejs/diagnostics/issues) will be created with the
date and time of the meeting. The issue will provide schedule logistics as well as an agenda,
links to meeting minutes, and information about how to join as a participant or a viewer.

#### Biannual F2F

Twice a year the diagnostics working group meets for a Face to Face gathering where current
concerns are hashed out in greater detail. An issue will be created for the event where
those who are interested may find details about how to participate.

See the [foundation calendar](https://nodejs.org/calendar) for meeting times.

### Members

<!-- ncu-team-sync.team(nodejs/diagnostics) -->

- [@AndreasMadsen](https://github.com/AndreasMadsen) - Andreas Madsen
- [@BridgeAR](https://github.com/BridgeAR) - Ruben Bridgewater
- [@cjihrig](https://github.com/cjihrig) - Colin Ihrig
- [@Flarna](https://github.com/Flarna) - Gerhard Stöbich
- [@gireeshpunathil](https://github.com/gireeshpunathil) - Gireesh Punathil
- [@harshithakp](https://github.com/harshithaKP) - Harshitha K P
- [@hashseed](https://github.com/hashseed) - Yang Guo
- [@hekike](https://github.com/hekike) - Peter Marton
- [@Hollerberg](https://github.com/Hollerberg) - Gernot Reisinger
- [@jasnell](https://github.com/jasnell) - James M Snell
- [@jkrems](https://github.com/jkrems) - Jan Olaf Krems
- [@joyeecheung](https://github.com/joyeecheung) - Joyee Cheung
- [@kjin](https://github.com/kjin) - Kelvin Jin
- [@legendecas](https://github.com/legendecas) - Chengzhong Wu
- [@mhdawson](https://github.com/mhdawson) - Michael Dawson
- [@mike-kaufman](https://github.com/mike-kaufman) - Mike Kaufman
- [@mmarchini](https://github.com/mmarchini) - Mary Marchini
- [@naugtur](https://github.com/naugtur) - Zbyszek Tenerowicz
- [@ofrobots](https://github.com/ofrobots) - Ali Ijaz Sheikh
- [@othiym23](https://github.com/othiym23) - Forrest L Norvell
- [@Qard](https://github.com/Qard) - Stephen Belanger
- [@rafaelgss](https://github.com/RafaelGSS) - Rafael Gonzaga
- [@sam-github](https://github.com/sam-github) - Sam Roberts
- [@watson](https://github.com/watson) - Thomas Watson
- [@yunong](https://github.com/yunong) - Yunong Xiao


<!-- ncu-team-sync end -->

#### Emeritus

- [@bnoordhuis](https://github.com/bnoordhuis) - Ben Noordhuis
- [@brycebaril](https://github.com/brycebaril) - Bryce Baril
- [@dberesford](https://github.com/dberesford) - Damian Beresford
- [@Fishrock123](https://github.com/Fishrock123) - Jeremiah Senkpiel
- [@piscisaureus](https://github.com/piscisaureus) - Bert Belder
- [@pmuellr](https://github.com/pmuellr) - Patrick Mueller
- [@rmg](https://github.com/rmg) - Ryan Graham
- [@thekemkid](https://github.com/thekemkid) - Glen Keane
- [@thlorenz](https://github.com/thlorenz) - Thorsten Lorenz
- [@trevnorris](https://github.com/trevnorris) - Trevor Norris
- [@danielkhan](https://github.com/danielkhan) - Daniel Khan
- [@mcollina](https://github.com/mcollina) - Matteo Collina
