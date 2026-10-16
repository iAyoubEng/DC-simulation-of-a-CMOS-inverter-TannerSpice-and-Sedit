# CMOS Inverter design — Schematic, Netlist & DC Simulation

A static CMOS inverter (NOT gate) designed and simulated in **Tanner EDA S-Edit** (schematic/symbol capture + netlist generation) and **T-Spice / W-Edit** (SPICE simulation + waveform viewing).

## Circuit

Standard CMOS inverter topology:

- **PMOS (M2)** — pull-up network, source tied to `Vdd`
- **NMOS (M1)** — pull-down network, source tied to `Gnd`

## Transistor sizing: why `Wn = 22 µm` and `Wp = 53 µm`

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



**Result.** The DC sweep (`simulation/vtc_waveform.png`) confirms it worked: input ramps `0 → 5 V` and the output crosses it at exactly **`VM = 2.5 V` = `Vdd/2`**, i.e. a perfectly centered voltage transfer curve. That symmetric crossing is the direct evidence of a balanced design — equal noise margins on both logic levels, and equal (symmetric) worst-case rise/fall drive strength.





## Tools

- Tanner **S-Edit** 7.03 — schematic & symbol capture, netlist export
- Tanner **T-Spice** — SPICE simulation
- Tanner **W-Edit** — waveform viewing

