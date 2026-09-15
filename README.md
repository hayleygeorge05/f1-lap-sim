# F1 Lap Time Simulator — Active Aero (2026 Regulations)

**Question:** How much lap time does 2026's active aerodynamics (with X-mode activated on the straights and Z-mode in the corners) save compared to a fixed-aero car, and does that time saved depend on the type of circuit?

**Answer:** Active aero reduces lap time by 2.34s at Zandvoort (a corner-dense circuit) but **9.93s** at Monza (a power circuit) — this is about 4x as much benefit where there is more straight-line distance to take advantage of reduced drag. This is in line with the physical reasoning that underlines the change to the regulations, namely that active aero becomes more advantageous the more of the lap is spent at high speed on the straights.

---

## Method

A point-mass lap time simulator built from real 2026 F1 telemetry
(via [FastF1](https://github.com/theOehrly/Fast-F1)):

1. A track model is obtained by calculating the corner curvature computed from the actual X/Y GPS position data and then         smoothing it using a Savitzky-Golay filter, giving a continuous radius of curvature at every point around the lap.
2. Vehicle model — a point-mass car with two aerodynamic states:
   - Z-mode (corners): high downforce, high drag
   - X-mode (straights): lower downforce, lower drag — the 2026 active-aero straight-line mode
3. The speed profile is obtained using the standard forward-backward pass technique: this involves setting a cornering-grip      limit at each point (determined numerically since downforce varies with speed), carrying out a forward pass to simulate       acceleration out of corners that is limited by power and traction, and then performing a backward pass to simulate braking    into corners. The speed at each point in the end is the smallest of these three constraints.
4. For validation, the simulated lap time for each circuit is compared with the actual qualifying pole lap for the circuit,      and the vehicle parameteres (such as tyre grip and drag coefficients) are adjusted until the speed profile of the model       matches the real telemetry to within ~5%.
5. Each circuit is run two times: one instance with active aero (the mode being switched according to the track curvature)       and another with a fixed-aero baseline (always in Z-mode, which represents a car of the pre-2026 type), thus providing a      direct measure of the time advantage gained from the active-aero. 

**Validation — simulated vs real speed trace:**

![Zandvoort validation](zandvoort_validation.png)
![Monza validation](monza_validation.png)

## Results

| | Zandvoort (corner-dense) | Monza (power circuit) |
|---|---|---|
| Real 2026 qualifying pole | 71.16s | 81.79s |
| Simulated (active aero) | 75.10s | 85.08s |
| **Validation gap** | +3.94s (5.5%) | +3.30s (4.0%) |
| Simulated (fixed aero) | 77.44s | 95.02s |
| **Active aero saving** | **2.34s** | **9.93s** |

**Active vs fixed aero speed traces:**

![Zandvoort active vs fixed](zandvoort_active_vs_fixed.png)
![Monza active vs fixed](monza_active_vs_fixed.png)

A real-world detail: Monza's real pole was Gasly's maiden F1 pole position — a true surprise result during the weekend when the data was collected.

## Modelling insight: track-specific aero setup

It was not possible to use the same vehicle parameters to validate the two circuits. Since Zandvoort is a high-downforce track and Monza has the lowest downforce setup of the season, each required its own calibration - this being a genuine and officially recorded engineering choice, namely that F1 teams use wing packages tailored to each track rather than sticking with one fixed setup for the entire season. At first, using the aero coefficients that had been calibrated at Zandvoort for Monza resulted in a 15% validation error; however, assigning Monza its own low-downforce baseline (which corresponded to a real low-drag wing package), the model was once again brought in line with the same ~4-5% standard of fit used at Zandvoort.

## Limitations

- There is a single global coefficient of friction. The model is not able to depict both a flowing corner (at Zandvoort) and    an extremely tight chicane (at Monza) using a single grip value - a known simplification since real tyres produce             considerably more grip at low speed than a flat coefficient suggests. 
- There is a track-loop boundary artefact since the curvature calculation has no awareness of the fact that a lap is a closed   loop, which results in a short and unrealistic speed spike right at the start/finish line in the raw model output.
- The model makes no allowance for thermal effects on the tyres, fuel load, or driver error, instead assuming idealised and     perfectly steady grip and braking throughout, which is one of the reasons why the simulated lap is consistently a bit         slower than the actual pole lap (a real driver and car exhibit variability which the model has no need to take into           account). 
- A simplified form of aero switching based on curvature. The system for switching to active aero is based on a threshold       corner radius rather than the specific track zone logic that real teams use.
- On long straight sections, the presence of numerical sensitivity occurs; on extremely long straights (especially at Monza),   the curvature is nearly zero, and this can cause small floating-point artefacts in the curvature calculation, requiring a     validated minimum radius to eliminate non-physical results. 

## Repository contents

- `lap_sim.ipynb` — full notebook: track modelling, vehicle model, speed profile calculation, validation, and active-vs-fixed    aero comparison for both circuits.
- `zandvoort_telemetry.csv`, `monza_telemetry.csv` — real 2026 qualifying telemetry used for track modelling and validation.

## Possible extensions

- Monte Carlo sensitivity analysis over aero and grip parameters, to give the lap time saving as a distribution rather than a   point estimate. 
- A third circuit type (e.g. a balanced circuit) to test whether the active-aero benefit scales smoothly with straight-line     proportion or behaves differently.
- A more detailed active-aero switching model based on defined track zones rather than a curvature threshold.
