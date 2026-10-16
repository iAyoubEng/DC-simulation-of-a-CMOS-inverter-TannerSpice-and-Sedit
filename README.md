# CMOS Inverter — Schematic, Netlist & DC Simulation

A static CMOS inverter (NOT gate) designed and simulated in **Tanner EDA S-Edit** (schematic/symbol capture + netlist generation) and **T-Spice / W-Edit** (SPICE simulation + waveform viewing).

## Circuit

Standard CMOS inverter topology:

- **PMOS (M2)** — pull-up network, source tied to `Vdd`
- **NMOS (M1)** — pull-down network, source tied to `Gnd`
- Gates tied together → input `X`
- Drains tied together → output `Y`

| | Schematic | Symbol |
|---|---|---|
| | `schematic/transistor_schematic.png` | `schematic/gate_symbol.png` |

## Netlist

Extracted SPICE netlist (`netlist/inverter.sp`):

```spice
M1 Y X Gnd Gnd NMOS L=2u W=22u AD=66p PD=24u AS=66p PS=24u
M2 Y X Vdd Vdd PMOS L=2u W=30u AD=66p PD=24u AS=66p PS=24u
```

| Transistor | L | W | W/L |
|---|---|---|---|
| NMOS (M1) | 2 µm | 22 µm | 11 |
| PMOS (M2) | 2 µm | 30 µm | 15 |

## Transistor sizing: why `Wn = 22 µm` and `Wp = 30 µm`

A plain inverter built with equal NMOS/PMOS widths is **not balanced**: holes move slower than electrons, so for the same W/L the PMOS pulls up weaker than the NMOS pulls down. That skews the switching threshold `VM` away from `Vdd/2`, which makes the logic-high and logic-low noise margins (`NMH` and `NML`) unequal — the circuit tolerates more noise on one logic level than the other.

**Balance condition.** At the switching threshold both transistors sit in saturation with equal current. Equating `ID,n = ID,p` and solving for `VM` gives:

```
VM = [ r·(VDD − |VTp|) + VTn ] / (1 + r),     where  r = √[ kp'(Wp/Lp) / kn'(Wn/Ln) ]
```

Setting `VM = VDD/2` (the symmetric point that makes `NML = NMH`) collapses this to `r = 1`, i.e.:

```
kn'·(Wn/Ln) = kp'·(Wp/Lp)   →   Wp/Lp = (µn/µp) · (Wn/Ln)
```

So the PMOS has to be made wider than the NMOS by the electron-to-hole mobility ratio (`µn/µp`) of the process, with everything else equal. Using the same channel length for both devices (`Ln = Lp = 2 µm`), this reduces to a simple width ratio:

```
Wp / Wn = µn / µp  →  Wp = 22 µm × (µn/µp) = 30 µm   →   µn/µp ≈ 1.36 for this process
```

**Result.** The DC sweep (`simulation/vtc_waveform.png`) confirms it worked: input ramps `0 → 5 V` and the output crosses it at exactly **`VM = 2.5 V` = `Vdd/2`**, i.e. a perfectly centered voltage transfer curve. That symmetric crossing is the direct evidence of a balanced design — equal noise margins on both logic levels, and equal (symmetric) worst-case rise/fall drive strength.

> `sizing-reference/reference_template_netlist.png` is the lab's original template netlist (`Wn=22u`, `Wp≈53u`, dated 2007) — included only for reference. It is **not** the sizing used in this design; the final values are `Wn=22µm / Wp=30µm` as shown in `netlist/inverter.sp` and reflected in the simulation above.

## Simulation

- **Sweep:** DC sweep of input `X` from 0 V to 5 V (`Vdd = 5 V`)
- **Plot:** Voltage Transfer Characteristic (VTC), `V(Y)` vs `V(X)`
- **Switching threshold:** `VM ≈ 2.5 V`

![VTC Waveform](simulation/vtc_waveform.png)

## Tools

- Tanner **S-Edit** 7.03 — schematic & symbol capture, netlist export
- Tanner **T-Spice** — SPICE simulation
- Tanner **W-Edit** — waveform viewing

## Repo structure

```
cmos-inverter-design/
├── README.md
├── schematic/
│   ├── transistor_schematic.png
│   └── gate_symbol.png
├── netlist/
│   └── inverter.sp
├── simulation/
│   └── vtc_waveform.png
└── sizing-reference/
    └── reference_template_netlist.png
```
