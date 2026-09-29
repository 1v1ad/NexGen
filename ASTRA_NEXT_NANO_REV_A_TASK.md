# Astra-6 Task — Smart AirKey NEXT Nano Rev.A

Status: **ratified initial engineering task**
Repository: **https://github.com/1v1ad/NexGen**
Primary source of product intent: **SMART_AIRKEY_NEXT_PRODUCT_VISION.md**

---

# 0. User authorization and repository authority

The repository owner explicitly authorizes Astra-6 to use **1v1ad/NexGen** for this project.

Astra is authorized to:

- clone/pull this repository on the user's PC;
- read all files in this repository;
- create project files under the repository workspace;
- create branches;
- commit project artifacts;
- push project branches to this repository;
- create documentation needed for the engineering work.

For normal work after this task file exists:

- do **not** force-push;
- do **not** rewrite published history;
- do **not** delete user-authored files unless a later explicit instruction permits it;
- prefer a dedicated working branch and a reviewable commit history;
- never touch unrelated repositories.

Suggested local workspace:

```text
C:\Projects\NexGen
```

If the repository already exists elsewhere on the machine, use the existing clean clone rather than creating conflicting copies.

At the start of every engineering run, record:

- exact repository path;
- branch;
- HEAD SHA;
- `git status --short`;
- KiCad version;
- OS/toolchain facts relevant to reproduction.

---

# 1. Mission

Design the first hardware prototype of **Smart AirKey NEXT Nano** in KiCad.

NEXT Nano Rev.A is a **single miniature combined reader + single-door access controller**.

It is not the future split Reader + Secure I/O product.

It is not the future battery reader.

Rev.A must combine, on one compact hardware platform:

- NFC/MIFARE reader;
- BLE;
- Wi-Fi;
- local access-control decision;
- secure device identity;
- local credential/policy state;
- protected lock output;
- Door Contact;
- REX;
- tamper;
- **Door Health current/voltage measurement**;
- local event journal;
- phone/cloud synchronization architecture;
- secure firmware/configuration update foundations.

The device is powered from typical access-control infrastructure, nominally **12/24 V DC**.

Primary product principle:

> **Reader. Controller. Cloud edge. One tiny device.**

## 1.1 Hard physical-design invariants

Two Rev.A design priorities are now explicit:

1. **Primary optimization target: minimum practical PCB/enclosure size** consistent with reliable NFC/RF performance, electrical robustness, thermal margin, field wiring robustness, manufacturability and first-prototype success.
2. **Rev.A MUST use a 4-layer PCB unless a later explicit human decision approves otherwise.** Saving a few dollars by reducing the layer count is **not** a design objective.

Miniaturization is a real product requirement, not decorative wording. Astra should actively look for area reductions through justified component/package selection, two-sided placement where appropriate, 0201 passives where they materially help, compact connectors/interconnect strategy, and careful functional integration.

However, size must never be “won” by sacrificing RF integrity, power robustness, Door Health measurement quality, thermal headroom, manufacturing yield, testability or safety.

---

# 2. Read before doing anything

Before modifying hardware files:

1. read this entire task;
2. read `SMART_AIRKEY_NEXT_PRODUCT_VISION.md`;
3. inspect repository state;
4. inspect local KiCad installation;
5. identify what manufacturer documentation is available or must be obtained;
6. create a documented engineering plan.

Do not begin PCB routing merely because KiCad is available.

---

# 3. Hard Rev.A boundary

## Included in Rev.A

Rev.A is the **monoblock NEXT Nano**.

It may include:

- one primary MCU;
- NFC;
- Wi-Fi/BLE;
- secure element;
- power conversion;
- protected lock driver;
- lock current/voltage sensing;
- door inputs;
- tamper;
- status indication;
- buzzer;
- production/debug interface;
- reasonable legacy interfaces if area/pin budget permits.

## Explicitly NOT Rev.A

Do not turn Rev.A into the future two-block product.

The following remain documented future variants only:

- NEXT Secure with outside Reader + protected inside Secure I/O;
- NEXT Air with battery-powered outside reader;
- low-power radio between reader and internal controller;
- two-year reader battery target;
- wireless power through the door;
- UWB premium proximity;
- full controller-to-controller mesh.

You may preserve architectural compatibility with those ideas, but **do not add components to Rev.A solely for them** unless there is a clear low-cost reason and it is documented.

---

# 4. Critical product behavior

## 4.1 Offline-first

Ordinary credential decisions must not depend on cloud availability.

The device must be capable of using the last valid local:

- credential data;
- schedule;
- access policy;
- configuration;
- required cryptographic state.

Network loss must not disable a normally authorized door.

## 4.2 Cloud/phone synchronization

Rev.A architecture should support:

- BLE commissioning;
- BLE Courier Sync for small payloads;
- temporary Wi-Fi/hotspot for larger synchronization and OTA;
- eventual direct facility Wi-Fi if configured.

A phone is transport, not root of trust.

Signed/configured authority remains independently verifiable by the device.

## 4.3 Deterministic access authority

Do not use AI to decide whether a person is allowed through the door.

Access authority is deterministic, local, signed/auditable and policy based.

AI may later analyze maintenance/anomaly telemetry only.

---

# 5. Door Health is mandatory

Door Health is not a future placeholder.

It is a **Rev.A flagship engineering requirement**.

The controller must not merely switch a lock.

It must measure what happens.

The design must support, at minimum:

- lock supply voltage measurement;
- lock current measurement;
- meaningful sampling of the lock current profile;
- detection or inference of open load;
- detection or inference of short circuit / overcurrent;
- undervoltage information;
- driver fault information where available;
- correlation with Door Contact timing;
- collection of data needed for future predictive maintenance.

The eventual firmware should be able to characterize:

- inrush/peak current where applicable;
- steady current;
- unlock-to-door-open delay;
- door-open duration;
- close-to-relatch delay;
- repeated abnormal electrical signatures.

Rev.A does not need ML.

Rev.A does need **good raw measurements and reliable hardware**.

---

# 6. Power and lock output

Input power target:

```text
Nominal systems: 12 V / 24 V DC
Engineering input target for investigation: approximately 10–30 V DC
```

The exact qualified range must be based on selected components.

Required protection study:

- reverse polarity;
- surge/transient behavior appropriate to access wiring;
- TVS strategy;
- input fuse/PTC/eFuse strategy if appropriate;
- brownout;
- bulk capacitance;
- regulator thermal margin.

## 6.1 Real-world poor-power requirement

Do **not** design Rev.A around a clean laboratory 12.000 V bench supply.

On real access-control sites the device may be powered from inexpensive 12 V or 24 V security power supplies, including common charger/backup units with a lead-acid battery. The quality of those supplies may be mediocre.

Phase A must therefore characterize and design for realistic field conditions including:

- DC input substantially above/below nominal within a defined qualified range;
- low-frequency ripple from inexpensive supplies;
- switching noise;
- startup overshoot;
- brownout and recovery;
- charger/battery switchover;
- momentary supply sag when the lock operates;
- shared-ground disturbances;
- transients coupled from long field wiring;
- inductive energy/noise from the lock;
- repeated power interruption and rapid restart.

Astra must propose a **qualified operating range and a separate transient-survival target**, supported by component ratings and margin. Do not merely write “10–30 V” if the selected protection/regulator chain cannot prove it.

Investigate appropriate combinations of:

- input TVS;
- reverse-polarity protection;
- eFuse/PTC/fuse where justified;
- input LC/π filtering where stable and appropriate;
- local bulk energy storage;
- UVLO with useful hysteresis;
- regulator headroom;
- brownout detection/reset;
- separation/filtering between noisy lock power and sensitive MCU/NFC/RF/measurement rails.

The prototype test plan must include intentionally poor/noisy supply conditions, not only nominal bench power.

Prefer avoiding a large electromechanical relay if a robust solid-state solution is practical.

Investigate:

- protected MOSFET;
- smart high-side switch;
- smart low-side switch;
- dedicated load driver.

Selection must account for:

- fail-safe locks;
- fail-secure locks;
- inductive loads;
- realistic current;
- thermal dissipation;
- fault handling;
- measurement strategy;
- package size.

Do not choose lock-current capability by guess.

Provide calculations and margins.

---

# 7. Door I/O

Minimum:

- Door Contact input;
- REX / Exit Button input;
- tamper input.

Prefer at least one spare protected configurable input if practical.

Inputs must be treated as field wiring, not clean laboratory GPIO.

Investigate:

- ESD;
- transients;
- pull topology;
- debounce;
- accidental wiring;
- NO/NC configuration;
- cable fault diagnostics if practical.

---

# 8. NFC / credential engineering

Rev.A must support a useful NFC/MIFARE path.

Conceptual credential priorities:

1. secure credential such as DESFire EV2/EV3 or equivalent;
2. smartphone NFC/BLE path;
3. legacy MIFARE/UID compatibility where commercially required.

UID-only access must not be presented as the secure design target.

Perform a documented trade study between at least:

- **PN7160-class NFC controller**
- **ST25R3916B-class NFC front-end**

These names are candidates, not predetermined winners.

For each option compare:

- package size;
- BOM;
- firmware complexity;
- supported credential modes;
- antenna requirements;
- matching network;
- reference designs;
- availability;
- lifecycle;
- integration risk;
- power;
- test complexity.

Use current manufacturer datasheets/reference designs/errata.

Do not trust remembered pinouts.

---

# 9. MCU / connectivity engineering

A compact ESP32-C6 variant with integrated flash is a candidate because Rev.A needs Wi-Fi + BLE in a very small space.

It is not automatically ratified.

Compare the selected MCU against requirements:

- package/PCB area;
- Wi-Fi;
- BLE;
- flash capacity;
- secure boot;
- signed OTA support;
- anti-rollback primitives;
- cryptographic acceleration;
- ADC suitability if used for Door Health;
- available GPIO;
- UART/SPI/I2C;
- debug/production flashing;
- temperature range;
- supply requirements;
- lifecycle/availability.

If a better architecture uses an external ADC/current monitor rather than MCU ADC, document why.

---

# 10. Secure element

Investigate **SE051/SE05x-class** or a current equivalent.

Goals:

- per-device hardware identity;
- private key protection;
- authenticated commissioning;
- future Reader ↔ Secure I/O pairing compatibility;
- protected signing/verification operations where appropriate.

Do not store production private keys as ordinary readable flash blobs.

The secure-element choice must be documented, not assumed.

---

# 11. Security foundations

Rev.A hardware/firmware architecture should allow:

- secure/verified boot;
- signed firmware;
- firmware anti-rollback;
- signed configuration;
- configuration anti-rollback;
- authenticated device identity;
- production debug policy;
- tamper event;
- persistent event integrity;
- safe recovery from interrupted update.

Do not create custom cryptographic algorithms.

Use well-established primitives and implementation libraries supported by the selected silicon.

---

# 12. Event journal / black-box design

Plan a durable local event journal.

The data model should accommodate:

- monotonic sequence;
- timestamp;
- event type;
- credential reference;
- policy version;
- decision;
- firmware/config version;
- Door Health metrics;
- fault reason.

Plan for tamper-evident chaining/checkpoints.

Do not implement “blockchain”.

The engineering goal is detectable history modification/deletion.

Hardware/flash endurance and power-loss behavior must be considered.

---

# 13. Time integrity

Offline schedules require time.

Explicitly design how Rev.A knows time.

Investigate:

- MCU RTC capability;
- external RTC if justified;
- backup strategy;
- time after long power loss;
- trusted time synchronization;
- monotonic counter independent of wall clock;
- behavior when time is unknown/untrusted.

A reset clock must not silently become a valid schedule decision.

Document the chosen failure behavior.

---

# 14. Atomic configuration / safe persistent state

Design a persistent-state scheme where an interrupted update cannot destroy the last valid access configuration.

Desired logic:

```text
receive candidate
→ verify signature
→ validate schema
→ validate device/site target
→ validate version
→ write inactive copy/slot
→ verify integrity
→ atomically activate
```

Exact implementation depends on flash layout and MCU.

Plan it before firmware grows.

---

# 15. User interface

Rev.A shall include provision for a **very small, modest-volume audible indicator**.

Buzzer requirements:

- SMD part preferred;
- physically small;
- clearly audible at the reader at short distance but not intended to be an alarm-class sounder;
- software-controllable;
- must support complete software disable;
- support quiet/hospital/night behavior;
- avoid unnecessary standby current;
- compare piezo vs magnetic SMD options on size, drive circuit, current and achievable SPL.

Rev.A shall also reserve hardware for visual indication.

Minimum intent:

- at least **two independently controllable LED indications**, or an equivalently flexible arrangement such as an RGB status LED plus a second independent indicator;
- firmware-controlled brightness/duty cycle;
- ability to turn all LEDs fully off;
- footprint/routing may be populated or DNP depending on enclosure/optical design;
- consider light-pipe / diffuser geometry and NFC/RF interaction.

Do not spend large PCB area on decorative lighting, but do not paint the product into a corner that prevents future front-panel illumination.

Optional haptic feedback may be investigated only if it is justified by area/power/product value.

Support product modes such as:

- normal;
- quiet/hospital;
- night;
- service;
- error/fault.

UI hardware must not compromise NFC/Wi-Fi antenna performance.

---

# 16. Legacy interoperability

Investigate whether Rev.A can economically expose any of:

- Wiegand input/output;
- OSDP / RS-485;
- generic protected I/O.

Do not let legacy compatibility destroy the Nano size target.

Priority:

1. standalone Nano functionality;
2. secure/future-compatible interface if practical;
3. legacy Wiegand compatibility.

Document pin/area/BOM trade-offs.

The broader “CAN-filter for access control” concept remains part of the product vision and may become a later SKU or firmware mode.

---

# 17. Physical / PCB target

Desired class: **very small**.

A useful investigation target is approximately **25–35 mm-class electronics PCB**, but this is not a hard dimension.

Do not produce a smaller but bad RF/power design.

RF, thermal, connector, installation and manufacturing constraints win.

Possible strategies:

- PCB NFC loop;
- separate FPC NFC antenna;
- front-face antenna larger than electronics area;
- ferrite or metal compensation where needed.

Explicitly assess installation near:

- metal door;
- metal door frame;
- glass;
- wood;
- plastic enclosure.

---

# 18. Passive/package policy

Preferred default:

- 0402 passives where sensible;
- **0201 explicitly allowed and encouraged** where it materially reduces critical PCB area or improves placement around dense IC/RF sections.

Rev.A manufacturing must **not** be constrained to low-cost “economy” assembly if that forces 0402 and makes the product materially larger. Use a production/standard PCBA service capable of reliable 0201 placement when needed.

Do not use 0201 everywhere merely because it is small. Favor it where the area gain is real and where values/ratings/availability are suitable.

For every proposed 0201-heavy region, verify the selected assembler's current:

- minimum package capability;
- spacing rules;
- stencil/paste requirements;
- AOI/inspection capability;
- component sourcing/feeder requirements;
- panel/fiducial requirements.

Do not assume that a fab which can manufacture the bare PCB can also assemble the chosen package mix economically.

QFN/BGA/WLCSP are allowed when justified.

Every footprint must be checked against the exact selected manufacturer's package drawing.

No invented footprint.

No “probably compatible”.

---

# 19. Stack-up

**Mandatory Rev.A default: 4-layer PCB.**

Astra must not reduce Rev.A to 2 layers as a cost optimization. A layer-count reduction requires a later explicit human approval after evidence that RF, return paths, power integrity, Door Health sensing and manufacturability are not compromised.

Initial conceptual structure:

1. signal/components;
2. solid ground;
3. power/secondary signal;
4. signal/components.

Final stack-up must be designed against a realistic board house capability.

Preserve:

- clean RF return paths;
- Wi-Fi keepout;
- NFC constraints;
- separation of noisy switching current from sensitive sensing/RF;
- Kelvin sensing where applicable;
- short lock-current path;
- thermal copper.

---

# 20. KiCad requirements

Use native KiCad project files.

Required repository organization target:

```text
hardware/
  next-nano/
    kicad/
    libraries/
    datasheets/
    manufacturing/

firmware/
  next-nano/
  shared/

docs/
  architecture/
  decisions/
  tests/
```

Do not add generated junk or machine-local caches to Git.

Create/update `.gitignore` when necessary.

Use project-local libraries for custom verified symbols/footprints.

---

# 21. Engineering evidence rules

For every critical part selected, record:

- exact manufacturer;
- exact manufacturer part number;
- datasheet revision/link/reference;
- package;
- supply range;
- temperature rating;
- current/voltage limits relevant to design;
- footprint source;
- key layout constraints;
- unresolved risk.

If a datasheet cannot be obtained, do not silently design from memory.

Mark it as a blocker or select a component whose documentation can be verified.

---

# 22. Workflow and gates

## Phase 0 — environment / repo reconnaissance

Do not change hardware yet.

Report:

- repository path;
- current branch;
- HEAD;
- Git status;
- KiCad version;
- available local EDA helpers/plugins;
- whether manufacturer docs can be downloaded/accessed;
- expected manufacturer/assembly workflow.

Create:

```text
docs/architecture/REV_A_ENVIRONMENT.md
```

---

## Phase A — architecture study

No PCB routing.

Produce:

```text
docs/architecture/REV_A_ARCHITECTURE_REVIEW.md
docs/decisions/COMPONENT_TRADE_STUDY.md
docs/architecture/POWER_BUDGET.md
docs/architecture/DOOR_HEALTH_ARCHITECTURE.md
docs/architecture/SECURITY_MODEL.md
docs/architecture/REV_A_BLOCK_DIAGRAM.md
```

At minimum determine/propose:

- MCU;
- NFC architecture;
- secure element;
- power input/protection;
- regulators;
- lock driver;
- current/voltage measurement;
- field input circuits;
- debug/programming;
- memory/flash partition concept;
- antenna concept;
- candidate connectors/terminals;
- preliminary PCB dimensions;
- thermal risks;
- BOM-risk items.

End the review with exactly these sections:

```text
BLOCKERS
IMPORTANT RISKS
OPEN DECISIONS
PROPOSED COMPONENTS
EXPECTED PCB SIZE
EXPECTED POWER/THERMAL LIMITS
NEXT GATE
```

### First-run stop gate

**For the first Astra run on this project, stop after Phase A.**

Commit and push the Phase 0 + Phase A artifacts to a dedicated branch.

Do **not** proceed into schematic capture until the human review explicitly ratifies the architecture.

This is deliberate.

---

## Phase B — schematic capture

Only after ratification.

Create native KiCad schematic.

Requirements:

- verified symbols;
- verified footprints;
- explicit net names;
- readable hierarchy;
- test points;
- power flags done correctly;
- no unexplained ERC waivers.

Subcircuits should include:

- input protection;
- power tree;
- MCU;
- NFC;
- secure element;
- RF/antenna interface;
- lock driver;
- Door Health measurement;
- field inputs;
- LED/buzzer/UI;
- debug/programming;
- legacy interfaces if ratified.

Deliver a schematic design review document.

---

## Phase C — PCB placement

Before routing, produce placement proposal.

Critical review areas:

- NFC antenna/matching;
- Wi-Fi antenna/keepout;
- buck converter current loops;
- secure element proximity/routing;
- ADC/current sensing;
- lock high-current path;
- TVS/protection placement;
- connector entry;
- thermal paths;
- test points;
- enclosure/mechanical assumptions.

Do not route until placement is internally reviewed.

---

## Phase D — routing

Then route.

Rules:

- respect stack-up;
- preserve ground plane integrity;
- minimize switching loops;
- protect analog measurement from noisy power;
- use Kelvin strategy for shunt if applicable;
- respect RF reference designs;
- verify differential/controlled impedance where actually required;
- do not create decorative impedance constraints without manufacturer need.

---

## Phase E — adversarial hardware review

After DRC:

Do not conclude “board ready”.

Run a separate review whose prompt is effectively:

> Find the reason this board will fail after fabrication.

Check at least:

- pin mapping;
- power pins;
- boot strapping;
- reset/enable;
- oscillator;
- decoupling;
- regulator stability;
- feedback component ratings;
- level compatibility;
- pull-ups/pull-downs;
- open-drain requirements;
- ADC range;
- current-sense polarity/range;
- load-driver dissipation;
- inductive transient;
- TVS selection;
- reverse polarity;
- ESD path;
- connector polarity;
- footprint pin-1;
- exposed-pad connection;
- thermal vias;
- Wi-Fi keepout;
- NFC tuning/matching;
- antenna metal interaction;
- testability;
- programming;
- manufacturing clearances.

Every waiver must have a written reason.

---

# 23. Firmware bring-up scope

Do not write the entire cloud product before hardware exists.

Initial firmware skeleton after hardware ratification should prove:

## Boot/security
- deterministic boot;
- watchdog;
- device identity;
- secure storage primitives;
- version reporting.

## NFC
- detect credential;
- read basic identifier/data;
- exercise secure credential path as hardware permits.

## Connectivity
- BLE advertising/service;
- Wi-Fi connect/disconnect;
- temporary hotspot credential workflow prototype.

## Door
- read Door Contact;
- read REX;
- read tamper;
- command lock safely;
- acquire lock voltage/current profile.

## Diagnostics
- report supply voltage;
- report lock profile;
- report fault reason;
- append event log record.

No cloud dependency for local door operation.

---

# 24. Door Health Rev.A test plan

Plan bench tests for:

- normal lock;
- disconnected lock;
- shorted/overloaded output under safe controlled conditions;
- lower/higher supply voltage within intended range;
- different representative lock loads;
- door contact immediate/open delay;
- stuck/held-open simulation;
- repeated cycles;
- thermal soak appropriate to prototype resources.

Record current profiles.

Define what Rev.A can reliably classify.

Do not claim predictive failure detection until data supports it.

---

# 25. Security test ideas

Rev.A engineering tests should eventually include:

- invalid config signature;
- stale config version;
- interrupted config write;
- interrupted firmware update;
- replayed network/courier payload;
- credential replay where protocol allows testing;
- tamper;
- MCU reboot during lock operation;
- brownout;
- corrupted event record;
- debug access state.

Future NEXT Secure will address physical separation attacks that Nano cannot fully eliminate.

Document that boundary honestly.

---

# 26. Manufacturing outputs — later gate

When the design is eventually ratified for prototype manufacturing, produce:

- KiCad source;
- clean ERC;
- clean DRC;
- BOM;
- CPL/Pick-and-Place;
- Gerber or agreed fabrication data;
- drill files;
- PDF schematic;
- fabrication notes;
- assembly notes;
- programming/test notes;
- board renders.

Do **not** place any order, pay a manufacturer, or send production files externally without an explicit human instruction.

---

# 27. Git workflow for Astra

For the first run, create a dedicated branch such as:

```text
astra/next-nano-rev-a-architecture
```

Before writing:

- verify branch/HEAD;
- verify clean status or report existing changes.

Commit logical artifacts.

Suggested first-run commits:

```text
docs: capture NEXT Nano Rev.A environment
docs: add Rev.A architecture and component trade study
```

Push the branch to **1v1ad/NexGen**.

Do not merge to main.

Report the branch and exact commit SHA(s) at the end.

---

# 28. Final first-run report format

When Phase A is complete, respond with:

```text
REPOSITORY
- local path:
- branch:
- base SHA:
- final SHA:
- git status:

KICAD
- version:
- relevant tools/plugins:

ARCHITECTURE
- MCU:
- NFC:
- secure element:
- input power:
- lock driver:
- Door Health measurement:
- field inputs:
- antenna concept:
- estimated PCB size:

BLOCKERS
- ...

IMPORTANT RISKS
- ...

OPEN DECISIONS
- ...

FILES CREATED
- ...

VALIDATION PERFORMED
- ...

RECOMMENDATION FOR PHASE B
- ...
```

Do not hide uncertainty.

Do not convert an assumption into a “fact” merely to finish the task.

---

# 29. Acceptance gate for Phase A

Phase A is acceptable only if a human reviewer can answer:

- What exactly are we building?
- Why these main ICs?
- Can it survive 12/24 V door wiring?
- How does it switch the lock?
- How does it measure Door Health?
- Is the current measurement actually within range and resolution?
- Where are the RF antennas and keepouts?
- What limits minimum board size?
- What are the major thermal risks?
- What ambient temperature range is qualified, and why?
- What happens on a cheap/noisy 12/24 V field PSU, brownout, charger/battery switchover and lock-induced supply sag?
- What happens offline?
- What is stored securely?
- What does Rev.A explicitly not solve?
- What would stop us from drawing the schematic now?

If those answers are not supported by manufacturer documentation and calculations, Phase A is not complete.

---

# 30. Core engineering rule

> **Do not optimize for impressiveness. Optimize for a board that will work on the first prototype.**

When miniaturization conflicts with:

- RF integrity;
- electrical robustness;
- thermal margin;
- security;
- serviceability;
- manufacturability;

document the conflict and prefer reliability unless the user explicitly chooses otherwise.

A tiny PCB is a product feature only after it works.

## 30.1 Temperature requirement

Rev.A is expected to be installed in real building environments and may be located near entrances where temperature can be substantially worse than a climate-controlled office.

Use the following **preliminary design target** unless Phase A identifies a documented blocker:

- target ambient operation: **-40 °C to +60 °C**;
- prefer critical semiconductors/components rated at least **-40 °C to +85 °C** (or better) where practical;
- perform worst-case regulator/lock-driver/PCB thermal analysis at the hot end;
- check oscillator, NFC matching-relevant parts, current-sense accuracy, protection devices and capacitors across temperature;
- do not assume electrolytic capacitance/ESR or ceramic effective capacitance is constant across temperature/bias;
- document any component that becomes the temperature-limiting item.

If the realistic Rev.A product class cannot honestly meet -40…+60 °C, report the exact limitation and evidence rather than silently relaxing the target.

---

# 31. Explicit reminder about future variants

Do not lose these ideas.

They remain in the product roadmap:

### NEXT Secure
External reader + protected internal Secure I/O.

### NEXT Air
Battery external reader + low-power authenticated air link to the internal controller, with predictive battery maintenance.

### Future slogan
**Wireless outside. Secure inside.**

But **NEXT Nano Rev.A is one combined controller/reader**.

Do not accidentally redesign it into two boxes during Phase A.
