### Exercise 1: SIR models

## Part 1 — Initial Conditions and Baseline Simulation
# Selecting and defending initial conditions
I choose a fixed population of \(N=135{,}000\) as a rounded approximation for a simplified New Haven community, rather than an exact census count. My initial conditions are:
- \(I(0)=1\): The scenario specifies that a single infectious index case enters New Haven on Day 0. I assume there are no other existing infections and include this individual in the modeled population of 135,000.
- \(R(0)=0\): I assume that no individuals have already recovered from this pathogen or have protective immunity at the start. Although a novel pathogen does not necessarily imply an absence of cross-immunity, no evidence of prior immunity is provided, so zero initial removal is a transparent simplifying assumption.
- \(S(0)=134{,}999\): All remaining individuals are assumed susceptible. Therefore, \(S(0)=N-I(0)-R(0)=135{,}000-1-0=134{,}999\).
These choices satisfy \(S(0)+I(0)+R(0)=N\) and represent the introduction of one infection into an otherwise fully susceptible community.

Here's the initial settings: 
Total population: 135,000
Initial susceptible population: 134,999
Initial infectious population: 1
Initial removed population: 0
Transmission rate: 0.3 per day
Removal rate: 0.1 per day
Mean infectious duration: 10.0 days

# complete time course of the infectious population over time 
<img width="984" height="484" alt="image" src="https://github.com/user-attachments/assets/39ee7bc6-56a7-4cc4-99c1-d44869eb01d6" />

## Part 2 - Step Size Validation and Policy Analysis
# Evaluating Step Size
Here's the result: 
| dt | t_peak | I_peak | peak_time_error_days | peak_size_error_pct | max_curve_error_pct |
|---:|---:|---:|---:|---:|---:|
| 1.00 | 68.00 | 42167.5263 | 5.025 | 3.9063 | 29.3181 |
| 0.50 | 65.50 | 41352.4705 | 2.525 | 1.8979 | 14.9876 |
| 0.10 | 63.40 | 40719.0579 | 0.425 | 0.3371 | 2.7370 |
| 0.05 | 63.15 | 40640.7683 | 0.175 | 0.1442 | — |

An illustrative version:
<img width="1084" height="584" alt="image" src="https://github.com/user-attachments/assets/3ad91f2a-5549-42b1-b499-a7bf82670e62" />

I assessed whether \(dt\) was sufficiently small by comparing peak timing, peak magnitude, and the full infectious-population curve with a 
finer Euler approximation.I selected a peak-timing tolerance of 0.5 days under an explicit assumption that the model would support daily 
resource planning: numerical timing error should remain below half of that planning interval. I also selected 1% tolerances for peak 
magnitude and maximum curve error, normalized by the reference peak, to require close numerical agreement relative to the outbreak size. 
These are illustrative numerical acceptance criteria, not established clinical standards; a different application could require tighter 
tolerances. I additionally checked that errors decreased as the step size was reduced. A candidate step size was accepted only when all 
three criteria were met.

# What happens if \(dt\) is too large?
An excessively large time step can produce inaccurate peak estimates, negative compartment populations, nonphysical oscillations, 
or numerical divergence. For example, an update may remove more susceptible individuals than are available. Because such results are 
biologically impossible, the solver reports an error rather than replacing negative values with zero. Population conservation alone is 
insufficient to establish accuracy because an inaccurate simulation can still preserve \(S+I+R=N\).

| dt | reference_dt | peak_time_error_days | peak_size_error_pct | max_curve_error_pct | passed |
|---:|---:|---:|---:|---:|:---|
| 0.050 | 0.006250 | 0.20625 | 0.168217 | 1.370810 | False |
| 0.025 | 0.003125 | 0.11875 | 0.084028 | 0.685948 | True |

Validated dt: 0.025000 days 
Peak timing: 63.05 days 
Peak magnitude: 40,602 individuals 
Operational clearance: day 201.38 
Maximum population conservation error: 0.00000000

# Informatics & Policy Analysis
Peak infection measures how many individuals are infectious simultaneously, whereas cumulative infection measures how many individuals have 
been infected over the entire outbreak. Hospital administrators must manage concurrent demand for beds, staff, and equipment. Two outbreaks 
with similar cumulative infections can create very different capacity pressures if one concentrates infections into a shorter period. However, 
infectious population is only a proxy for hospital demand; estimating actual occupancy also requires hospitalization probabilities, admission 
delays, and lengths of stay.

The infectious population changes according to

$$
\frac{dI}{dt}
= I\left(\beta\frac{S}{N}-\gamma\right).
$$

The peak occurs when new infections balance removals:

$$
\beta\frac{S}{N}=\gamma.
$$

Equivalently, the effective reproduction number reaches one:

$$
\mathcal{R}_{\mathrm{eff}}
=\frac{\beta}{\gamma}\frac{S}{N}=1.
$$

t this threshold, each infectious individual produces, on average, one new infection. Once the susceptible fraction falls below 
$1/\mathcal{R}_0$, each infectious individual produces fewer than one new infection, so removals exceed new infections and the infectious 
population declines. For the baseline $\mathcal{R}_0=3$, the peak occurs when approximately one-third of the population remains susceptible: 
$S(t)=135{,}000/3=45{,}000$. Therefore, the infection peak occurs while many susceptible individuals remain, rather than requiring the 
susceptible population to reach zero.

## Part 3 — Parameter Uncertainty and Sensitivity Analysis
| beta | gamma | reproduction_number | t_peak | I_peak |
|---:|---:|---:|---:|---:|
| 0.150000 | 0.05 | 3.000000 | 125.950 | 40582.260912 |
| 0.171429 | 0.05 | 3.428571 | 105.650 | 47135.161167 |
| 0.192857 | 0.05 | 3.857143 | 91.125 | 52784.298662 |
| 0.214286 | 0.05 | 4.285714 | 80.225 | 57695.960282 |
| 0.235714 | 0.05 | 4.714286 | 71.700 | 62003.247653 |

<img width="1469" height="584" alt="image" src="https://github.com/user-attachments/assets/c452fc7c-4960-4da1-befb-bc9c6c6a489a" />

At a fixed γ, sensitivity to β is indicated by how quickly the color changes horizontally in the right heatmap. Peak infections are particularly sensitive 
in the low-to-moderate β range, approximately β = 0.18–0.30 with γ = 0.08–0.12, where increases in β produce noticeable increases in \(I_{\max}\).

In contrast, \(I_{\max}\) is relatively insensitive to additional increases in β in the lower-right region, approximately β = 0.35–0.45 and γ = 0.05–0.07. 
Transmission already substantially exceeds removal there, so further increases in β produce smaller changes in peak magnitude. Near β ≈ γ in the upper-left 
corner, peak infections remain small in absolute terms, although their percentage changes can still be large.

Masks and social distancing directly reduce β by decreasing effective transmission between individuals. At the baseline γ = 0.10, moving left from β = 0.30 in the 
heatmaps generally delays the infection peak and reduces its magnitude. A 50% reduction in β, from 0.30 to 0.15, moves the model into a region with a substantially 
later and smaller peak, providing more preparation time and reducing concurrent demand.

To prevent initial epidemic growth altogether, the intervention must satisfy:
\[
\mathcal{R}_{\mathrm{eff}}(0)
=\frac{\beta_{\mathrm{new}}}{\gamma}\frac{S(0)}{N}\leq1.
\]
Because the population is initially almost entirely susceptible, this requires approximately β ≤ 0.10, or a 66.7% reduction from baseline. 
This threshold lies outside the plotted β range. At or below this threshold, infections decline from the start, so the intervention prevents a 
growing outbreak rather than simply delaying its peak.

However, the heatmaps cannot establish exactly which reduction would prevent hospital overload. That requires hospital capacity, hospitalization 
probabilities, and lengths of stay. The appropriate intervention must reduce expected hospital demand below available capacity; delaying the 
peak alone does not guarantee this outcome.

