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

### Exercise 2: In Silico Synergy Optimization & Resource-Constrained Experimental Design
## Part 1: API Architecture & Ethical Query Restrictions
# RESTful service design for  https://ramcdougal.com/cgi-bin/error_function.py?a=0.4&b=0.2Links to an external site

The endpoint does not follow a clean, resource-oriented URL design because /cgi-bin/error_function.py exposes the server’s implementation technology and names a computational script rather than a resource. However, using query parameters such as a and b is not itself a violation of REST principles, and GET is appropriate if the request only retrieves a simulated score without changing server state. The URL alone is insufficient to determine whether the complete service satisfies all REST constraints.

Standardized endpoints make computational biology pipelines easier to integrate and maintain because they provide consistent parameter names, response formats, units, and error handling across assay platforms.

# Reduction and query efficiency

The principle of Reduction maps to minimizing unnecessary API evaluations while retaining enough information to make a reliable optimization decision. My pipeline will cache previously evaluated dose pairs, reuse existing scores when estimating gradients, apply explicit stopping criteria, and enforce a shared query budget across all starting points. Multiple starting points will help explore different basins without performing an exhaustive grid search. In a real experiment, however, repeated measurements may still be necessary to quantify biological variability.

# User-Agent and request delays

A custom User-Agent identifies the automated client and helps the server distinguish legitimate coursework requests from unidentified automated traffic. It is not universally required by HTTP, but it may be required by this service’s access policy. The pipeline will send requests sequentially with a configurable delay and will respect rate-limit responses, including Retry-After when provided. Failed requests will also count toward the request budget, preventing retries from creating uncontrolled traffic.

## Part 2: Custom 2D Gradient Descent Engine & Experimental Defense


# 1. Finite Difference Derivative Estimation

Because the API returns only the disease burden $E(a,b)$, I estimate the gradient numerically:

$$
\nabla E(a,b)
\approx
\left(
\frac{E(a+h,b)-E(a,b)}{h},
\frac{E(a,b+h)-E(a,b)}{h}
\right).
$$

I use $h=10^{-4}$ in the normalized dosage domain. This is a practical initial choice that provides a small perturbation without approaching machine precision. It is not a universally optimal step size: its accuracy depends on the smoothness and numerical precision of the API response.

The current score is reused, so a gradient estimate requires at most two additional API requests. This is more query-efficient than central differences, which generally require four perturbed evaluations for two parameters, although one-sided differences have first-order truncation error rather than second-order error. Near an upper boundary, the algorithm uses a backward difference to keep all queries feasible.

This approach assumes a deterministic, sufficiently smooth simulation. For noisy biological measurements, very small perturbations may be overwhelmed by experimental variability, requiring larger steps or replicate measurements.

# 2. Learning Rate Choice

The algorithm uses projected gradient descent:

$$
\mathbf{x}_{k+1} = \Pi_{[0,1]^2}\left(\mathbf{x}_k - \alpha_k \nabla E(\mathbf{x}_k)\right).
$$

where $\mathbf{x}_k=(a_k,b_k)$ and $\Pi$ projects the update onto the permitted dosage domain.

I start with a learning rate of $\alpha=0.1$. Because the objective's scale and curvature are unknown, this value is an initial trial rather than a guaranteed stable rate. I also cap the Euclidean length of each proposed update at 0.1, which limits abrupt changes in the normalized dosages.

The algorithm evaluates each proposed point and accepts it only if it reduces the burden and meets a sufficient-decrease condition. Otherwise, it halves the learning rate and tries again, with at most eight trials per iteration. Following a successful update, the next trial learning rate increases by 20%, up to a maximum of 0.25.

This rule balances faster progress against the risk of overshooting a narrow minimum. Backtracking adds evaluation costs, so the number of trials is limited and all requests count toward the total budget. These controls improve numerical reliability but do not guarantee that a narrow optimum will be found.

# 3. Dosage Bounds

Both dosages must satisfy:

$$
0\leq a\leq1,\qquad 0\leq b\leq1.
$$

I enforce these constraints by projecting each proposed update onto the feasible domain:

$$
a_{\mathrm{new}} = \min(1,\max(0,a_{\mathrm{proposed}})).
$$

$$
b_{\mathrm{new}} = \min(1,\max(0,b_{\mathrm{proposed}})).
$$

For example, a proposed dosage of $a=1.04$ becomes $a=1$, while $b=-0.03$ becomes $b=0$. The API client also checks all coordinates before sending a request, and finite-difference probes remain within the same bounds.

# 4. Convergence and Termination Criteria

I use several complementary stopping rules:

- **Projected gradient residual:** Stop when
$$
\left\|
\mathbf{x}-\Pi_{[0,1]^2}
\left(\mathbf{x}-\nabla E(\mathbf{x})\right)
\right\|_2 \leq 10^{-3}.
$$
  Unlike the ordinary gradient norm, this measure can recognize a constrained optimum at a boundary, where the gradient may point outside the feasible domain. The tolerance is an illustrative numerical criterion and does not establish biological or clinical optimality.

- **Small improvements:** Stop after three consecutive accepted updates improve the score by no more than $10^{-6}\max(1,|E|)$. Requiring consecutive small improvements reduces the chance of stopping after a single unproductive step, although this remains a heuristic.

- **Query budget:** Allow at most 200 attempted API requests across the client. Cached evaluations do not require new requests, while failed HTTP requests still count toward the limit.

- **Iteration and backtracking limits:** Stop after 60 iterations or if no improving step is found within eight backtracking trials.

These limits are explicit computational choices rather than experimentally established standards. They prevent indefinite spending when each evaluation represents a costly cohort. The program reports the stopping reason because reaching a budget or backtracking limit does not prove convergence.

A single run identifies a candidate minimum within the starting point's basin of attraction; it does not establish that the result is the global minimum. Multiple starting points will be examined in Part 3.

## Part 3: Non-Convex Fitness Landscapes & Multi-Start Strategy

# Optimization Results and Classification

The five trajectories reached two distinct candidate minima:

| Classification | Compound A dosage | Compound B dosage | Disease burden |
|:---|---:|---:|---:|
| Global-minimum candidate | 0.711929 | 0.168905 | ≈1.000000 |
| Local-minimum candidate | 0.216127 | 0.689268 | ≈1.100000 |

The lower-burden candidate was reached from starting points $(0.40,0.20)$ and $(0.75,0.25)$. The remaining three starting points approached the higher-burden candidate. This demonstrates why a single starting point is insufficient: a trajectory can approach a locally optimal combination while missing a better combination elsewhere.

The lower minimum has approximately 9.1% less disease burden than the higher minimum:

$$
\frac{1.1-1.0}{1.1}\times100\%\approx9.1\%.
$$

# Numerical Validation

| Candidate | Projected gradient residual | Neighborhood check passed |
|:---|---:|:---|
| Lower-burden minimum | 0.000301 | True |
| Higher-burden minimum | 0.000728 | True |

Both projected gradient residuals were below the selected tolerance of $10^{-3}$. Neither candidate had a lower-scoring point among the eight tested neighbors at coordinate perturbations of 0.002. These results support approximate local stationarity and local minimality at the tested scale.

Given the assignment's stated landscape containing a local and a global minimum, the lower-burden candidate is identified as the estimated global minimum and the higher-burden candidate as the estimated local minimum. However, finite multi-start searches and neighborhood checks do not constitute a mathematical proof of global optimality.

# Query Efficiency and Stopping Reasons

The search used 153 API requests, including 24 requests from Part 2, before candidate validation. Four trajectories stopped after satisfying the projected gradient tolerance. The trajectory starting at $(0.25,0.25)$ exhausted its allocated per-run query allowance, so that trajectory alone should not be described as converged. However, two other trajectories independently approached the same higher-burden minimum and satisfied the gradient criterion.

Reusing the Part 2 result, sharing cached evaluations, and allocating a limited query allowance to each additional start allowed exploration of multiple basins within the overall 200-request budget.

