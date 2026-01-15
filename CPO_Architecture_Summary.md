# CPO Switch Architecture Summary

## Overview

In Co-Packaged Optics (CPO) switch architecture, the **ELS (External Light Source)** provides continuous wave (CW) light exclusively to the **Tx path**. The Rx path operates independently without requiring the local ELS.

---

## Key Concepts

| Term | Definition |
|------|------------|
| **CPO** | Co-Packaged Optics - optical engine integrated on switch package |
| **ELS** | External Light Source - provides CW laser light |
| **CW** | Continuous Wave - unmodulated, stable carrier light |
| **OE/PIC** | Optical Engine / Photonic IC - contains modulators and photodetectors |
| **Power Fiber** | Delivers CW light from ELS to OE (Tx only) |
| **Media Fiber** | Carries modulated data light between endpoints |

---

## Tx Architecture

```mermaid
flowchart LR
    subgraph HostASIC[Host ASIC]
        SerDesTx[SerDes Tx]
    end

    subgraph OE[OE/PIC on Package]
        Driver[Driver]
        Mod[Modulator<br/>MZM/Ring]
    end

    subgraph External[Front Panel]
        ELS[ELS Module<br/>CW Laser]
    end

    SerDesTx -->|Electrical<br/>PAM4| Driver
    Driver -->|Electrical| Mod
    ELS -->|CW Light<br/>Power Fiber| Mod
    Mod -->|Modulated<br/>Data Light| MediaTx[Media Fiber]
    MediaTx --> RemoteRx[Remote Rx]
```

**Tx Signal Flow:**
1. ASIC SerDes generates baseband electrical signal
2. Driver amplifies signal to OE modulator
3. ELS provides CW light via power fiber
4. Modulator imprints data onto CW light
5. Modulated data light exits via media fiber

---

## Rx Architecture

```mermaid
flowchart LR
    RemoteTx[Remote Tx] --> MediaRx[Media Fiber]

    subgraph OE[OE/PIC on Package]
        PD[Photodetector<br/>PD]
        TIA[TIA]
    end

    subgraph HostASIC[Host ASIC]
        SerDesRx[SerDes Rx]
    end

    MediaRx -->|Modulated<br/>Data Light| PD
    PD -->|Current| TIA
    TIA -->|Electrical| SerDesRx
```

**Rx Signal Flow:**
1. Remote Tx sends modulated data light
2. Media fiber delivers light to local OE
3. Photodetector converts light to current
4. TIA amplifies and converts to voltage
5. SerDes Rx processes electrical signal

---

## Complete System View

```mermaid
flowchart TB
    subgraph System[CPO Switch System]

        subgraph Control[Management]
            HC[Host Controller]
        end

        subgraph FrontPanel[Front Panel - Pluggable]
            ELS[ELS Module]
        end

        subgraph Package[On-Package]
            OE[OE/PIC]
        end

        subgraph ASIC[Switch ASIC]
            TX[SerDes Tx]
            RX[SerDes Rx]
        end

    end

    HC <-->|CMIS| ELS
    HC <-->|CMIS| OE
    ELS -->|CW Light| OE
    TX <-->|Electrical| OE
    RX <-->|Electrical| OE
    OE <-->|Data Light| Fiber[Media Fiber to Remote]
```

---

## ELS Role Summary

| Aspect | Tx Path | Rx Path |
|--------|---------|---------|
| **Uses ELS?** | Yes | No |
| **Light Source** | CW from ELS | Data light from remote |
| **OE Function** | Modulator | Photodetector + TIA |
| **Fiber Type** | Power + Media | Media only |

---

## Key Takeaways

- ELS outputs **CW light** (carrier) with no data modulation
- CW light flows only to **Tx modulators** via power fiber
- Rx uses **incoming data light** from remote, independent of local ELS
- Host controller manages both ELS and OE via **CMIS protocol**
- Separating laser (ELS) from package improves **thermal management** and **serviceability**
