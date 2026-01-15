# CPO Switch Architecture Summary with Real Examples

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
        SerDesTx[SerDes Tx<br/>200G/lane PAM4]
    end

    subgraph OE[OE/PIC on Package]
        Driver[Driver]
        Mod[Modulator<br/>MZM/Ring]
    end

    subgraph External[Front Panel]
        ELS[ELS Module<br/>CW Laser<br/>0-10dBm]
    end

    SerDesTx -->|Electrical<br/>Baseband| Driver
    Driver -->|Amplified| Mod
    ELS -->|CW Light<br/>Power Fiber| Mod
    Mod -->|Modulated<br/>Data Light| MediaTx[Media Fiber<br/>SMF 2-10km]
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
    RemoteTx[Remote Tx] --> MediaRx[Media Fiber<br/>SMF 2-10km]

    subgraph OE[OE/PIC on Package]
        PD[Photodetector<br/>PD]
        TIA[TIA]
    end

    subgraph HostASIC[Host ASIC]
        SerDesRx[SerDes Rx<br/>CDR + Equalization]
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

## Real-World Example: NVIDIA Spectrum-X / Quantum-X

```mermaid
flowchart TB
    subgraph NV[NVIDIA CPO Switch - 51.2T]

        subgraph FP[Front Panel]
            ELS1[ELS #1]
            ELS2[ELS #2]
            ELSN[ELS #N]
        end

        subgraph PKG[On-Package Silicon Photonics]
            PIC1[PIC #1<br/>8T8R]
            PIC2[PIC #2<br/>8T8R]
            PIC3[PIC #3<br/>8T8R]
            PIC4[PIC #4<br/>8T8R]
        end

        subgraph ASIC[Spectrum ASIC]
            SerDes[128x 200G<br/>SerDes]
        end

        ELS1 -->|36 Power<br/>Fibers| PIC1
        ELS1 -.->|CW<br/>Fan-out| PIC2
        ELS1 -.->|1:4 Split| PIC3
        ELS1 -.->|Per ELS| PIC4

        SerDes <-->|Electrical| PIC1
        SerDes <-->|Electrical| PIC2
        SerDes <-->|Electrical| PIC3
        SerDes <-->|Electrical| PIC4

        PIC1 <-->|144 Media<br/>Fibers| Remote1[Remote<br/>Endpoints]
        PIC2 <-->|Data| Remote1
        PIC3 <-->|Data| Remote1
        PIC4 <-->|Data| Remote1
    end

```

**NVIDIA Configuration:**
- 1 ELS → 4 PICs (8T8R each)
- Each PIC: 8 Tx lanes × 200G = 1.6T
- CW fan-out: 1 CW source → 8 modulators per PIC
- Total: 324 fibers (144 media + 36 laser power)
- Switch capacity: 51.2T

---

## Real-World Example: Broadcom Tomahawk CPO

```mermaid
flowchart TB
    subgraph BCM[Broadcom Tomahawk - 25.6T]

        subgraph PKG[On-Package Integrated]
            ELS_INT[Integrated<br/>Laser Array]
            OE1[OE #1<br/>400G-FR4]
            OE2[OE #2<br/>400G-FR4]
            OEN[OE #N<br/>16 total]
        end

        subgraph ASIC[Tomahawk ASIC]
            SerDes2[64x 400G<br/>SerDes]
        end

        ELS_INT -->|On-Die<br/>CW| OE1
        ELS_INT -.->|CWDM 4λ| OE2
        ELS_INT -.->|Integrated| OEN

        SerDes2 <-->|Electrical| OE1
        SerDes2 <-->|Electrical| OE2
        SerDes2 <-->|Electrical| OEN

        OE1 <-->|16 Fiber<br/>Pairs| Remote2[Remote<br/>Endpoints]
        OE2 <-->|CWDM| Remote2
        OEN <-->|Data| Remote2
    end

```

**Broadcom Configuration:**
- Integrated laser array on package (not external ELS)
- 400G-FR4 per OE (CWDM 4λ)
- 16 fiber-pairs per engine → 6.4T per OE
- Total: 16 OE × 400G = 6.4T per direction
- Switch capacity: 25.6T

---

## Architecture Comparison

```mermaid
flowchart TB
    subgraph Arch[CPO Architecture Variants]

        subgraph Type1[Type 1: External ELS]
            T1_ELS[ELS Module<br/>Pluggable]
            T1_OE[OE/PIC]
            T1_ASIC[ASIC]

            T1_ELS -->|Power Fiber| T1_OE
            T1_ASIC <-->|Electrical| T1_OE
        end

        subgraph Type2[Type 2: Integrated Laser]
            T2_Laser[Laser Array<br/>On-Package]
            T2_OE[OE/PIC]
            T2_ASIC[ASIC]

            T2_Laser -->|On-Die CW| T2_OE
            T2_ASIC <-->|Electrical| T2_OE
        end
    end

    Type1 -.->|Example| NV[NVIDIA<br/>Spectrum-X]
    Type2 -.->|Example| BCM[Broadcom<br/>Tomahawk]

```

| Feature | NVIDIA Spectrum-X | Broadcom Tomahawk |
|---------|-------------------|-------------------|
| **Laser Type** | External ELS (pluggable) | Integrated on-package |
| **PIC Config** | 8T8R per PIC | 400G-FR4 per OE |
| **CW Delivery** | Power fiber (36 fibers) | On-die waveguide |
| **Serviceability** | ELS replaceable | Full package replacement |
| **Thermal** | Laser isolated from ASIC | Laser shares package thermal |
| **Capacity** | 51.2T | 25.6T |

---

## ELS Role Summary

| Aspect | Tx Path | Rx Path |
|--------|---------|---------|
| **Uses ELS?** | Yes (NVIDIA) / Integrated (Broadcom) | No |
| **Light Source** | CW from ELS or on-die laser | Data light from remote |
| **OE Function** | Modulator | Photodetector + TIA |
| **Fiber Type** | Power + Media (NVIDIA) / Media only (Broadcom) | Media only |

---

## Key Takeaways

- **NVIDIA approach**: External ELS allows independent laser thermal management and field replacement
- **Broadcom approach**: Integrated laser simplifies fiber management but requires full package replacement
- Both architectures use CW light **only for Tx path**
- Rx always uses **incoming modulated data light** from remote endpoints
- Choice depends on tradeoffs: serviceability vs integration density
