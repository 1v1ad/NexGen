# Smart AirKey NEXT — Product Vision & Engineering Ideas

Status: **living concept / product vision**
Repository: **1v1ad/NexGen**
Initial target: **NEXT Nano Rev.A**
Working slogan for future secure split architecture: **Wireless outside. Secure inside.**

> This document intentionally captures the broad idea space before optimization cuts it down.  
> It is not a PCB specification and not a promise that every feature belongs in Rev.A.

---

## 1. Why this product should exist

The goal is not to make “another NFC reader”.

The goal is to build a very small access-control edge device that combines the roles that are usually split between a reader, a door controller, a network gateway, an event logger and part of a diagnostic system.

The first product is **Smart AirKey NEXT Nano**: a **combined reader + single-door controller** in one miniature device.

The long-term family can then evolve into more security-oriented split architectures without throwing away the protocol, cloud model, firmware architecture or hardware concepts developed for Nano.

The product should be attractive for:

- premium residential buildings;
- hospitals and clinics;
- offices;
- technical rooms;
- elevators and lift equipment;
- distributed facilities where Ethernet is undesirable or impossible;
- retrofit projects where an existing door must gain cloud/phone capabilities without a large controller cabinet;
- installations where maintenance cost matters as much as access control itself.

The central idea is:

> **Reader. Controller. Cloud edge. One tiny device.**

Rev.A should prove that this concept can be physically small, robust, secure, offline-first and manufacturable.

---

# 2. Product family

## 2.1 NEXT Nano — Rev.A priority

A monolithic device.

One compact PCB/module contains:

- NFC/MIFARE reader;
- BLE;
- Wi-Fi;
- secure identity;
- access-control logic;
- local credential/policy storage;
- door inputs;
- lock driver;
- lock current/voltage telemetry;
- tamper;
- local event journal;
- cloud synchronization;
- phone-assisted synchronization;
- firmware update capability.

It is powered from typical access-control power: nominal **12/24 V DC**.

This is the first hardware target.

### Important security limitation

Because Nano contains the lock output, a Nano installed on the untrusted side of a door cannot provide the same physical-attack isolation as the future two-part Secure architecture.

Rev.A should still include reasonable defenses:

- secure boot;
- secure element;
- encrypted secrets;
- anti-rollback;
- tamper event;
- protected outputs;
- no trivial “short these two terminals to open” design;
- conformal coating / potting readiness where appropriate;
- audit logging.

But the architecture itself must not pretend that a monoblock mounted outside is equivalent to a protected internal controller.

That limitation is explicit and becomes the reason for NEXT Secure.

---

## 2.2 NEXT Secure — future

Two physical blocks:

### Outside
**Nano Reader**

Contains user interaction:

- NFC/MIFARE/DESFire;
- mobile credential;
- status LED / halo;
- buzzer;
- optional haptic;
- tamper;
- cryptographic identity.

It **cannot electrically open the door**.

### Inside
**Secure I/O Controller**

Located on the protected side near the lock.

Contains:

- final access decision;
- access policy;
- lock output;
- Door Health measurements;
- door contact;
- REX;
- event log;
- network/cloud functions;
- verified link to the external reader.

The design principle:

> **Wireless outside. Secure inside.**

“Wireless outside” refers primarily to the user/cloud interaction experience; the first Secure variant may still use a protected wired link between Reader and Secure I/O.

This architecture protects against the classic problem where ripping an outside reader off the wall exposes door-opening wiring.

---

## 2.3 NEXT Air — future

An evolution of NEXT Secure.

The outside reader has **no cable through the premium door at all**.

The external reader:

- is battery powered;
- sleeps almost all the time;
- wakes on NFC/card presence or local interaction;
- uses a very low-power authenticated radio link to the powered internal controller;
- reports battery health and predicted replacement date.

The internal controller:

- remains mains/access-power supplied;
- performs final GRANT/DENY;
- controls the lock;
- performs Door Health;
- handles cloud/BLE/Wi-Fi;
- can scan for phones without consuming reader battery.

Design target for suitable traffic profiles: roughly **2+ years battery life**, to be proven by a real power budget and prototype measurements, not by marketing assumptions.

Future battery telemetry should estimate remaining service life using:

- elapsed time;
- operation count;
- temperature history;
- open-circuit voltage;
- voltage under a known load;
- estimated internal resistance;
- model of the specific cell chemistry.

Cloud UI should be able to say:

> Replace battery on Door 173 within 90 days.

Not merely “battery = 32%”.

---

## 2.4 NEXT Air Power — experimental future

For suitable non-metal doors, investigate an external reader powered without a conventional cable, e.g. controlled inductive power transfer through the door panel.

This is exploratory only.

Do not contaminate Rev.A with this requirement.

---

# 3. Rev.A system philosophy

## 3.1 Offline-first

Internet is never required to make an ordinary local access decision.

The device stores the last valid:

- credential state;
- schedule;
- policy;
- device configuration;
- cryptographic material required for normal operation.

If the site has no Internet for days or weeks, the door must continue to operate.

Cloud connectivity is used for:

- event upload;
- policy/config update;
- time correction;
- device health;
- Door Health telemetry;
- OTA firmware;
- administrative operations.

Cloud outage must not turn a valid door into a brick.

---

## 3.2 The phone is a courier, not root of trust

The product should work in locations with no permanent Ethernet and possibly no permanent Wi-Fi.

An authorized smartphone can temporarily become a transport bridge.

Small data:

**Controller → BLE → Phone → Cloud**

and

**Cloud → Phone → BLE → Controller**

Large data / OTA:

**Phone hotspot → temporary Wi-Fi → Controller**

The phone should not need access to plaintext secrets or the ability to forge configuration.

Desired model:

- controller exports sealed/encrypted event bundles;
- phone transports them;
- cloud reads them;
- cloud sends signed config/policy bundles;
- phone transports them back;
- controller verifies them locally.

A compromised courier phone must not be able to invent a valid policy update.

Temporary hotspot credentials should not be retained longer than necessary.

---

# 4. Credential architecture

Support a migration path, not one credential forever.

## 4.1 Legacy
MIFARE Classic / UID based use may be supported for compatibility where customers already have cards.

But UID alone is **not** treated as a secure credential.

## 4.2 Preferred secure card path
Use a cryptographically authenticated credential such as **MIFARE DESFire EV2/EV3 or equivalent**.

The intended property:

Cloning an observed UID is insufficient to obtain access.

## 4.3 Smartphone
Support a mobile credential architecture through NFC/BLE.

The exact credential protocol is not locked at concept stage.

## 4.4 Adaptive authentication
The same person may require different proof at different doors or times.

Examples:

- normal resident entrance: trusted phone / convenient mode;
- technical room: NFC credential;
- sensitive room: NFC + trusted phone;
- very sensitive workflow: two distinct authorized people / escort.

A risk or anomaly engine may later **increase** required authentication strength.

It must not independently grant a permission that deterministic access policy denies.

---

# 5. Deterministic signed policy engine

A core product idea is to separate “firmware logic” from “site access rules”.

Policy should be data, not arbitrary executable code.

Example rules:

```text
credential.role == doctor
AND schedule == duty
AND door.zone == surgery
→ grant
```

```text
credential.type == contractor
AND 08:00 < local_time < 18:00
AND escort_present
→ grant
```

```text
fire_mode == true
→ emergency_policy_3
```

Properties:

- deterministic;
- bounded CPU/time/memory;
- no arbitrary network access;
- no filesystem/code execution;
- schema validated;
- versioned;
- signed by an authority;
- anti-rollback;
- locally executable offline.

A future policy update should not require a firmware update.

Rev.A may implement only a deliberately small rule set/interpreter, but the storage/versioning/security architecture should not block the full policy engine later.

---

# 6. Atomic configuration

A broken or interrupted synchronization must not destroy the currently valid configuration.

Desired process:

1. receive candidate bundle;
2. verify signature;
3. verify schema;
4. verify target device/site;
5. verify monotonically acceptable version;
6. write inactive slot;
7. integrity check;
8. atomic activation;
9. retain safe rollback/recovery semantics without permitting unauthorized downgrade.

Power loss at any intermediate step must leave the last known-good configuration usable.

This same discipline should eventually be used for firmware metadata and other critical persistent state.

---

# 7. Secure device identity

Each device should have a hardware-protected identity.

Candidate component class: secure element such as **NXP SE05x / SE051 or an equivalent current part**.

The final choice must be verified against current manufacturer documentation before schematic capture.

Desired uses:

- device private key;
- proof of genuine device identity;
- cloud enrollment;
- authenticated commissioning;
- protection of high-value secrets;
- pairing future Reader ↔ Secure I/O;
- protected cryptographic operations.

Private keys should not simply exist as readable blobs in ordinary MCU flash.

---

# 8. Zero-touch commissioning

Installation should feel like a modern product, not a 2005 access controller.

No required workflow based on:

- fixed 192.168.x.x address;
- default admin/admin password;
- DIP-switch identity;
- laptop with vendor utility;
- mandatory USB-UART terminal.

Desired flow:

1. installer powers device;
2. authorized phone establishes NFC/BLE commissioning session;
3. phone reads/verifies factory public identity;
4. cloud reports:
   - serial;
   - genuine / not genuine;
   - claimed / unclaimed;
   - hardware revision;
5. installer chooses:
   - site;
   - building;
   - entrance;
   - door;
6. cloud creates signed provisioning bundle;
7. device validates and activates it.

Secure administrative re-claim must be possible when a device is moved or replaced.

Development/debug pads may exist on the PCB but are not part of end-user installation.

---

# 9. Cryptographic black box

The device should function as a small forensic recorder.

Each significant event records, where applicable:

- monotonic sequence number;
- timestamp;
- event type;
- credential reference;
- decision;
- policy version;
- firmware version;
- device state;
- relevant Door Health metrics;
- reason code.

Events should be tamper-evident.

Possible structure:

**event[n] contains hash of event[n-1]**

plus periodic authenticated checkpoints.

Events of special interest:

- boot;
- brownout;
- watchdog reset;
- firmware update;
- firmware rollback attempt;
- invalid configuration signature;
- enclosure tamper;
- repeated malformed credential;
- repeated malformed network traffic;
- lock open circuit;
- lock short circuit;
- abnormal lock current;
- forced-open door;
- held-open door;
- door failed to open after valid grant.

The goal is not blockchain theatre.

The goal is to make silent deletion/modification of individual historical records detectable.

---

# 10. Door Health — flagship feature

This is one of the most important differentiators.

A traditional access controller knows:

> I asserted the relay.

NEXT should know:

> I commanded the lock, measured what electrically happened, observed what mechanically happened through the door contact, and can tell whether the door behaved normally.

The lock output should therefore be instrumented.

## 10.1 Measure

At minimum:

- supply voltage;
- lock current;
- peak/inrush current where relevant;
- steady-state current;
- duration;
- output fault flags;
- driver temperature if available;
- door-contact timing.

## 10.2 Detect immediately

- no-load / broken wire;
- short circuit;
- overcurrent;
- undervoltage;
- thermal fault;
- supply collapse;
- commanded unlock with no expected load signature.

## 10.3 Learn a normal signature

For each door, firmware can gradually establish normal ranges such as:

- normal current;
- normal transient profile;
- normal unlock-to-open delay;
- normal open duration;
- normal close-to-relatch delay.

No ML is required for Rev.A.

Simple statistics and bounded thresholds are sufficient to prove the concept.

## 10.4 Predict maintenance

Over time detect trends:

- lock current slowly rising;
- mechanical opening time increasing;
- closing taking longer;
- intermittent electrical contact;
- PSU becoming marginal;
- coil behavior changing;
- door contact disagreeing with commanded behavior.

Instead of:

> Door 173 is broken.

the system can eventually say:

> Door 173 shows increasing unlock current and 41% slower close-to-relatch time over 12 days. Inspect lock/door mechanics.

This turns the controller into a maintenance sensor, not only a security device.

For property management, hospitals and large portfolios this may be a separate commercial value proposition.

---

# 11. Door state machine

Do not implement door control as one boolean relay.

Model explicit states such as:

- LOCKED;
- ACCESS_GRANTED;
- UNLOCKING;
- UNLOCKED;
- OPEN;
- CLOSING;
- RELOCKING;
- FAULT.

Events:

- forced open;
- held open;
- unlock failed;
- relock failed;
- lock electrical fault;
- door contact fault;
- REX;
- tamper;
- network unavailable;
- invalid credential;
- valid credential / policy denied;
- policy unavailable / integrity failure.

The state machine should be deterministic and testable.

---

# 12. Power output concept

Avoid a large electromechanical relay in the Nano if a protected solid-state solution is practical.

Investigate:

- protected MOSFET;
- smart high-side switch;
- smart low-side switch;
- dedicated protected load driver.

Selection criteria:

- 12/24 V door systems;
- expected lock currents;
- inductive load behavior;
- fail-safe vs fail-secure lock use;
- short-circuit tolerance;
- thermal protection;
- diagnostics;
- current-sense method;
- size;
- heat dissipation;
- reverse polarity and surge conditions.

A tiny PCB that overheats at realistic lock current is a failed design even if KiCad DRC is green.

---

# 13. Inputs

Minimum desired inputs:

- door contact;
- REX / exit button;
- tamper.

Prefer at least one spare general-purpose protected input if pin/area budget permits.

Inputs should tolerate real installation abuse better than bare MCU pins.

Consider:

- ESD;
- long cable transients;
- accidental wiring;
- debounce;
- configurable normally-open/normally-closed semantics;
- diagnostics where practical.

---

# 14. NFC/RF concept

Potential NFC directions to compare during engineering:

- NFC controller such as **PN7160-class**;
- RF front-end such as **ST25R3916B-class**.

These are candidates, not ratified parts.

The engineer/agent must check current datasheets, errata, package availability and reference layouts before selection.

Key constraints:

- 13.56 MHz antenna dominates physical product design;
- “small MCU” does not guarantee a small good reader;
- antenna geometry, Q, tuning, nearby metal and enclosure matter;
- Wi-Fi/BLE antenna needs its own RF keepout;
- the product must not sacrifice usable card/phone performance merely to achieve an impressive PCB dimension.

Possible mechanical strategies:

- PCB perimeter NFC loop;
- separate FPC NFC antenna;
- electronics PCB smaller than the front-face antenna;
- ferrite/metal mitigation where required by enclosure/installation.

RF performance must be measured on prototypes.

---

# 15. Connectivity

## 15.1 Wi-Fi
No mandatory Ethernet.

Wi-Fi may be used:

- when facility Wi-Fi is available;
- temporarily through an authorized phone hotspot;
- for OTA and larger transfers.

## 15.2 BLE
Used for:

- commissioning;
- phone credential / nearby interaction;
- Courier Sync;
- diagnostics;
- service mode.

## 15.3 No-cloud dependency
Ordinary access must continue without both.

---

# 16. Legacy gateway idea — “CAN filter for access control”

A long-term differentiator is making NEXT useful during retrofit, not only greenfield deployment.

Potential roles:

```text
NFC/BLE NEXT → legacy Wiegand controller
```

```text
legacy reader → NEXT → cloud
```

```text
NEXT → OSDP Secure Channel controller
```

```text
NEXT Nano → standalone lock
```

The device can become a security/telemetry gateway around legacy infrastructure.

Possible future functions:

- observe legacy transactions;
- detect invalid/unexpected sequences;
- log them;
- translate protocols;
- gradually migrate sites to secure credential transport.

Do not invent proprietary cryptography merely to differentiate.

Prefer established secure protocols when they satisfy requirements.

---

# 17. OSDP and Wiegand

Wiegand should be supported only as a legacy compatibility path if feasible.

OSDP Secure Channel is preferred for secure reader/controller integration in future split architectures.

For Nano Rev.A, physical I/O budget may force prioritization.

The decision must be made explicitly in a component/pin/area trade study.

---

# 18. Secure boot, OTA and recovery

Firmware must have a security lifecycle.

Desired properties:

- verified/secure boot;
- signed firmware;
- anti-rollback;
- A/B or otherwise power-loss-safe update strategy where flash allows;
- recovery image or safe recovery path;
- version reporting;
- update audit event;
- no unsigned production update path.

Debug interfaces must have a defined production-state policy.

Rev.A prototypes may need broad debug access; production must not inherit prototype trust assumptions accidentally.

---

# 19. Time integrity

Offline access schedules require trustworthy local time.

The architecture should explicitly decide:

- MCU RTC only vs external RTC;
- what happens after long power loss;
- trusted cloud time synchronization;
- monotonic counters independent of wall clock;
- behavior when wall clock is uncertain.

A device should not silently grant scheduled access based on nonsense time after a clock reset.

The policy engine should have a defined “time uncertain” state.

---

# 20. Store-and-forward / neighborhood sync — future

A controller that has received a signed configuration bundle may potentially relay that immutable bundle to nearby authorized controllers.

Candidate transports:

- BLE;
- 802.15.4 where supported;
- other low-energy local transports.

A relay node should not need authority to modify the bundle.

This can let a large property gradually synchronize even if only some devices see an Internet-connected phone.

Not Rev.A critical.

---

# 21. UWB — future premium

UWB may become useful for:

- more reliable proximity;
- distance bounding;
- reducing BLE-RSSI ambiguity;
- premium hands-free access;
- resistance to some relay/proximity attacks.

Do not put UWB on Rev.A merely because it is exciting.

Leave an architectural path for NEXT Premium / Rev.B.

---

# 22. User experience

The hardware should communicate clearly with minimal visible hardware.

Potential elements:

- RGB LED / halo;
- buzzer;
- optional haptic motor/driver if it materially improves experience;
- configurable feedback profiles;
- low-noise / hospital mode;
- night mode;
- accessibility-friendly patterns.

The user should not need to understand whether an access decision came from local policy, phone relay or cloud synchronization.

---

# 23. Mechanical/product ideas

NEXT Nano should feel like a product, not a bare PCB in a generic junction box.

Goals:

- very small frontal area;
- premium enclosure options;
- hidden mounting where possible;
- tamper detection;
- no exposed service connector;
- realistic assembly tolerances;
- serviceable enough for Rev.A;
- future potting/conformal-coating readiness.

A target PCB dimension in the ~25–35 mm class is interesting, but not an invariant.

RF performance, thermal margin, power wiring and manufacturability win over vanity dimensions.

---

# 24. Component scale

Desired passive default:

- 0402 where practical;
- 0201 welcomed when it genuinely reduces critical area and assembly capability supports it.

Do not choose tiny packages merely for aesthetics.

QFN/BGA/WLCSP are allowed if:

- routing is viable;
- assembly is realistic;
- inspection/rework strategy is understood;
- thermal and RF behavior are acceptable.

---

# 25. PCB stack

Rev.A should strongly consider **4 layers**.

Reasons:

- 2.4 GHz RF;
- 13.56 MHz NFC;
- switching power;
- current measurement;
- lock current;
- compact geometry;
- need for good return paths and ground control.

A likely conceptual stack:

1. signal/components;
2. solid GND;
3. power / secondary signal;
4. signal/components.

Final stack must match the selected manufacturer.

---

# 26. Candidate functional blocks for NEXT Nano

Not yet a BOM.

```text
10–30 V DC input
  ↓
protection / reverse polarity / TVS
  ↓
buck / power tree
  ├── MCU + Wi-Fi/BLE
  ├── NFC controller/front-end
  ├── secure element
  ├── indicators/buzzer
  ├── protected inputs
  └── smart lock driver
        ├── voltage measurement
        ├── current measurement
        └── protection/fault telemetry
```

Possible MCU candidate: compact ESP32-C6 variant with integrated flash.

Possible secure element candidate: SE051-class.

Possible NFC candidates: PN7160-class vs ST25R3916B-class.

These names are starting points for datasheet-based trade study, **not instructions to copy a remembered reference design**.

---

# 27. Security threat ideas

Threats to consider from the beginning:

- cloned/weak credential;
- replay;
- stolen phone;
- malicious courier phone;
- malicious Wi-Fi;
- forged cloud response;
- config rollback;
- firmware rollback;
- reader/controller physical tamper;
- debug-port abuse;
- flash extraction;
- glitch/brownout;
- malformed packet causing lock behavior;
- denial of service;
- memory corruption;
- event deletion;
- time manipulation;
- lock-wire short/open;
- attacker replacing the reader;
- legacy Wiegand sniff/injection.

Each threat should have an explicit disposition:

- prevented;
- detected;
- limited;
- accepted in Rev.A;
- solved only by NEXT Secure.

---

# 28. Reliability philosophy

The unit controls a real door.

Therefore:

- no cloud-only access decision;
- no unsafe default after corrupted config;
- no “if parser crashes, unlock” behavior;
- no silent downgrade;
- watchdog;
- brownout handling;
- persistent-state integrity;
- deterministic recovery;
- explicit safe modes.

Fail-safe vs fail-secure behavior is not one global answer; it depends on lock type and life-safety integration.

The product architecture must make that configuration explicit.

---

# 29. Fire/life-safety integration

Future deployment in hospitals, commercial sites and elevators means emergency behavior matters.

Rev.A should at least reserve a path for a dedicated, deterministic emergency/fire input or integration mechanism whose behavior does not depend on cloud availability.

The access-control product must not improvise life-safety behavior via AI.

Final implementation requirements depend on the target certification/jurisdiction and must be handled as an engineering/compliance workstream.

---

# 30. AI boundary

AI can help:

- engineering;
- anomaly triage;
- maintenance trend analysis;
- fleet diagnostics;
- summarizing logs;
- suggesting investigations.

AI must not be the uncontrolled authority that decides whether a person has access.

Access authority remains deterministic, signed and auditable.

---

# 31. Door Health business value

Door Health should be treated not just as a diagnostic checkbox but as a potential product tier.

Possible fleet dashboard:

- doors with rising current;
- doors with abnormal unlock time;
- doors with intermittent contact;
- PSU voltage degradation;
- high forced-open rate;
- high held-open duration;
- battery status for future NEXT Air;
- maintenance priority.

A large customer could use NEXT for preventive maintenance even if basic badge access is already solved.

---

# 32. Rev.A priorities

Rev.A should prove the fundamentals:

1. compact monoblock hardware;
2. NFC credential path;
3. BLE and Wi-Fi;
4. offline local decision;
5. secure device identity;
6. protected lock output;
7. Door Health acquisition;
8. door contact / REX / tamper;
9. durable event log;
10. safe configuration update architecture;
11. basic phone commissioning / Courier Sync;
12. manufacturable KiCad design.

It does **not** need to implement every future cloud feature before the first PCB is built.

---

# 33. Rev.A non-goals

Do not let these delay first hardware unless explicitly promoted:

- battery-powered external reader;
- two-board NEXT Secure;
- UWB;
- full mesh;
- inductive through-door power;
- sophisticated ML;
- giant general-purpose policy language;
- production certification;
- every legacy protocol;
- every lock type;
- custom cryptographic primitives.

---

# 34. Product evolution principle

The first PCB should not try to contain the whole product roadmap.

But Rev.A architecture should avoid gratuitously blocking the next products.

Shared concepts across Nano / Secure / Air should include:

- device identity;
- signed policy;
- cloud object model;
- event format;
- Door Health model;
- commissioning;
- secure update;
- credential abstraction;
- state machine;
- telemetry;
- manufacturing identity.

This is how NEXT Nano becomes the first member of a family rather than a disposable prototype.

---

# 35. Working slogans / product language

### NEXT Nano
**Reader. Controller. Cloud edge. One tiny device.**

### NEXT Secure / NEXT Air
**Wireless outside. Secure inside.**

### Door Health
**The controller should know not only that it switched the lock — it should know whether the door actually behaved correctly.**

---

# 36. Engineering rule

> Do not optimize for impressiveness. Optimize for a board that works on the first prototype.

Whenever miniaturization conflicts with:

- RF integrity;
- electrical robustness;
- thermal margin;
- security;
- serviceability;
- manufacturability;

document the trade-off and choose reliability unless an explicit product decision says otherwise.

A beautiful 18×18 mm board that fails in a metal door installation is not innovation.

A slightly larger board that works reliably, measures its lock, survives installation abuse and can evolve into NEXT Secure is.
