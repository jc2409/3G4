# 3G4 Medical Imaging & 3D Computer Graphics — Ultimate Study Notes
*Cambridge Engineering Tripos Part IIA | Compiled from 2010–2024 past papers and cribs*

---

## HOW TO USE THIS DOCUMENT

This document covers every topic that has ever appeared in 3G4 exams from 2010 to 2024, with model answers drawn directly from the official cribs. The 2026 prediction section at the end tells you exactly where to focus.

**Exam format:** 4 questions, answer all. Three topic pillars:
- **Medical Imaging** (X-ray CT, MRI, Ultrasound, Nuclear Medicine)
- **Curves, Surfaces & Interpolation** (Splines, Marching Cubes, Interpolation methods)
- **3D Graphical Rendering** (Phong model, Shading, Z-buffer, Ray tracing)

---

# PART 1: MEDICAL IMAGING

---

## 1A. X-ray CT & Iterative Reconstruction (AART)
*Appeared: 2010, 2021*

### Core Physics
- **Beer-Lambert law:** I = I₀ e^(−μx), where μ = linear attenuation coefficient
- **Measurement:** Q = −ln(I/I₀) = sum of μ along the X-ray path
- **For a ray through N pixels:** Q = Σ μᵢ (each pixel assumed uniform)

### AART Algorithm
**Equation governing AART:**
```
μᵢ_new = μᵢ_old + λ × (Q_measured − Q_estimated) / N_pixels_on_ray
```
- Q_estimated = Σ μᵢ_old along ray
- λ = relaxation factor (0 < λ ≤ 2)
- N = number of pixels the ray passes through
- The error (Q_measured − Q_estimated) is distributed equally across all pixels on the ray

**Role of relaxation factor λ:**
- λ = 1: standard update
- λ < 1: slower convergence but more stable
- λ > 1: faster convergence but may overshoot/oscillate
- λ = 2: maximum useful value

**Optimal ordering for fastest convergence:** Use most different angles first (e.g., alternate between projections separated by 90° rather than adjacent angles). This ensures each update provides maximally new information.

### Why Iterative > Filtered Backprojection
- Handles incomplete/sparse data (few projections)
- Can model exact detector geometry
- Can incorporate prior knowledge (e.g., image must be non-negative)
- Better noise handling in low-dose CT
- Can model polychromatic beam effects

### Polychromatic X-ray Problem (Beam Hardening)
- Real X-ray sources emit spectrum of energies
- Low-energy photons are absorbed preferentially (higher μ at low energy)
- As beam passes through tissue, it becomes "harder" (higher average energy, lower μ)
- Results in: cupping artefacts (edges appear brighter than centre in uniform object)
- **Fix:** Pre-filter beam with aluminium/copper to remove low-energy photons; calibration correction; iterative reconstruction that models the spectrum

### Sample Worked AART (2021 style)
Given 4 hexagonal rods with unknown μ, projections Q at 3 angles, initial μ = 5 for all rods, λ = 1:

1. For each ray: compute Q_estimated = Σ μᵢ_current along ray
2. Compute error = Q_measured − Q_estimated
3. Update: Δμ = λ × error / N, add Δμ to all rods on that ray
4. Repeat for all rays in one direction, then next direction
5. Continue until convergence (within tolerance of true values)

**Key:** Show all arithmetic for first direction's update. The final answer gives the true attenuation coefficients.

---

## 1B. Radon Transform & Sinograms
*Appeared: 2010, 2013, 2014, 2016, 2018*

### Definition
The 2D Radon transform maps f(x,y) to line integrals at angle φ and offset s:

```
R[f](s,φ) = ∫₋∞^(+∞) f(s cosφ − l sinφ, s sinφ + l cosφ) dl
```

- s = perpendicular distance of line from origin
- φ = angle of projection direction (measured anticlockwise from x-axis)
- l = coordinate along the integration line

**Projection at angle φ:** the function p(s) = R[f](s,φ) for fixed φ

**Sinogram:** the 2D function p(s, φ) for all s and 0 ≤ φ ≤ π; each row is one projection

### Key Results

**Strip of width w, value v (axis-aligned):**
```
R = wv / sin θ    (for θ ≠ 0)
R → ∞             (for θ = 0, for s within the strip)
```
This means two strips with the same product wv have IDENTICAL Radon transforms for all θ ≠ 0. They differ only at θ = 0 (different widths over which R → ∞).

**Important implication (2016):** This shows it is NOT impossible to determine width from CT — we can use the θ = 0 projection (direct measurement of width), or the range of s values over which R → ∞.

**Disk of radius r, value 1:**
```
R[f](s,φ) = 2√(r² − s²)   for |s| ≤ r
R[f](s,φ) = 0               for |s| > r
```
(Independent of φ by rotational symmetry)

**Point source δ(x−a)δ(y−b):**
```
R(s,φ) = δ(s − a cosφ − b sinφ)
```
Sinogram is a sinusoidal curve: s = a cosφ + b sinφ

**Diamond f(x,y) = 1 for |x|+|y| ≤ 1:**
- At φ = 0: projection is triangular, width 2, height 2 (peak at s=0)
- At φ = π/4: projection is rectangular, constant value √2 for |s| ≤ 1/√2
- Sinogram outline: half-width d(φ) = cos(φ) for 0 ≤ φ ≤ π/4, then symmetrically increases to 1 at φ = π/2

### Sample Q&A

**Q (2010):** Calculate the Radon transform of f(x,y) = 1 if √(x²+y²) ≤ 4, 0 otherwise.

**A:** By rotational symmetry, R is independent of φ. At offset s, the chord length through the disk is 2√(r²−s²) where r=4. Therefore:
```
R[f](s,φ) = 2√(16 − s²)   for |s| ≤ 4
R[f](s,φ) = 0               for |s| > 4
```

**Q (2014):** At s=0, φ=π/8, calculate R[f] where f(x,y) = 1 for |x|+|y| ≤ 1.

**A:** The chord intersects the diamond at two points. At φ=π/8, the perpendicular to the projection direction has angle π/8+π/2 = 5π/8. For the diamond |x|+|y|≤1, the intersection length at s=0:
- Distance from centre to edge in direction π/4: 1/cos(π/4) = √2 per side from origin
- At angle π/8 from the diagonal, distance d₁ = (1/√2)/cos(π/4 − π/8) = (1/√2)/cos(π/8)
- Total R = 2d₁ = √2/cos(π/8) ≈ **1.531**

**Q (2018):** Sketch the sinogram for f(x,y) = δ(x−2)δ(y−2) + δ(x)δ(y−1).

**A:** Each point source traces a sinusoid s = a cosφ + b sinφ in the sinogram.
- Source at (2,2): s = 2cosφ + 2sinφ = 2√2 sin(φ + π/4)
- Source at (0,1): s = 0·cosφ + 1·sinφ = sinφ
Two sinusoidal curves in the (s, φ) plane.

---

## 1C. Magnetic Resonance Imaging (MRI)
*Appeared: 2010, 2015, 2019, 2023*

### Fundamental Physics
- Proton (hydrogen nucleus) has spin → magnetic dipole moment
- In external field B₀, spins align parallel or anti-parallel → net magnetisation M₀ along z
- **Gyromagnetic ratio:** γ = 42.58 MHz/T for protons
- **Larmor frequency:** f = γB₀ (precession frequency when disturbed)
- At 1.5T: f = 63.9 MHz; at 3T: f = 127.7 MHz; at 5T: f = 212.9 MHz

### Relaxation Times
| Property | Description | Typical order |
|----------|-------------|---------------|
| T₁ | Spin-lattice: recovery of longitudinal Mz | Longest (hundreds ms to seconds) |
| T₂ | Spin-spin: decay of transverse Mxy from random interactions | Medium |
| T₂* | T₂ + field inhomogeneity decay | Shortest |

**T₁ recovery:** Mz(t) = M₀(1 − e^(−t/T₁))
**T₂ decay:** Mxy(t) = M₀ e^(−t/T₂)
**T₂* decay:** much faster than T₂; dominated by static field inhomogeneities

**Ordering (longest to shortest):** T₁ > T₂ > T₂*

### Free Induction Decay (FID)
1. Apply 90° RF pulse at Larmor frequency → tips Mz into xy-plane
2. Spins precess and emit signal at Larmor frequency
3. Signal decays rapidly with time constant T₂* (inhomogeneities cause dephasing)

**Why FID alone cannot give T₁, T₂, PD:**
- T₂* is dominated by static field inhomogeneities (magnet imperfections, susceptibility variations), not material properties
- Signal decays before material-dependent T₂ effects become apparent
- Cannot separate T₁, T₂, PD from single rapidly-decaying signal

### Spin-Echo Sequence
1. 90° RF pulse → spins in xy-plane, begin precessing and dephasing
2. Wait TE/2
3. 180° RF pulse → flips all spins (reverses phase evolution)
4. Spins refocus → echo forms at time TE
5. Echo amplitude ∝ e^(−TE/T₂) (T₂*, not T₂*, cancels out!)

**Key:** Spin-echo measures T₂, not T₂*. The 180° pulse undoes the static inhomogeneity dephasing.

### Image Weighting
| Type | TR | TE | Bright tissue |
|------|----|----|---------------|
| T₁-weighted | Short (< T₁) | Short (< T₂) | Fat (short T₁) |
| T₂-weighted | Long (> T₁) | Long (~ T₂) | Water, CSF (long T₂) |
| PD-weighted | Long | Short | High proton density |

**T₁-weighted mechanism:** Short TR → materials with shorter T₁ recover more longitudinal magnetisation between pulses → stronger signal → appear brighter. Fat (T₁ ≈ 200 ms) recovers most → brightest.

**T₂-weighted mechanism:** Long TE → materials with longer T₂ retain more transverse magnetisation → stronger echo → appear brighter. Water (T₂ ≈ 2000 ms) → brightest.

### Spatial Localisation (2023 focus)
MRI uses three gradient coils, each adding a linear field variation:
```
∂Bz/∂z = Gz,   ∂Bz/∂x = Gx,   ∂Bz/∂y = Gy
```

**Slice selection:**
- Apply Gz simultaneously with RF pulse
- Only protons at z where B = f/γ are at resonance and get excited
- Slice position: z = (f_RF − γB₀)/(γGz)
- Slice thickness ∝ bandwidth of RF pulse / Gz
- Rectangular slice → sinc-shaped RF pulse (in time); practical compromise: windowed sinc

**Phase encoding (y direction):**
- Apply Gy briefly after RF pulse
- Protons precess faster/slower → phases encode y-coordinate
- Switch off Gy → protons return to same frequency, but different phases remain
- Repeat sequence with different Gy values (k-space filling)

**Frequency encoding (x direction):**
- Apply Gx during signal readout
- Different x-positions precess at different frequencies → frequency encodes x
- Fourier transform of readout signal gives x-profile

**k-space encoding:** Repeat spin-echo with different Gy → fills k-space. 2D Fourier transform gives full image.

### Sample Q&A

**Q (2015):** Proton precession rate at 5T?
**A:** f = γB₀ = 42.58 × 5 = **212.9 MHz**

**Q (2015):** Describe T₁ relaxation.
**A:** T₁ (spin-lattice relaxation) describes the recovery of longitudinal magnetisation after an RF pulse. When spins are tipped into the transverse plane by a 90° pulse, they gradually return to alignment with B₀. Energy is transferred from the spin system to the surrounding atomic/molecular environment (the "lattice"). The recovery follows: Mz(t) = M₀(1 − e^(−t/T₁)). T₁ depends on how efficiently the lattice absorbs energy at the Larmor frequency.

**Q (2023):** How are gradient fields used for spatial encoding?
**A:** Gradient coils apply linear field variations in each principal direction. The z-gradient Gz selects a slice by ensuring only protons at z = (f_RF − γB₀)/(γGz) are at resonance during the RF pulse. After the RF pulse, the y-gradient Gy is applied briefly to impose phase differences on spins according to y-position — this is phase encoding. During readout, the x-gradient Gx is applied so different x-positions precess at different frequencies — this is frequency encoding. By repeating the sequence with different Gy values, k-space is filled, and a 2D Fourier transform reconstructs the image.

---

## 1D. Ultrasound Imaging
*Appeared: 2015, 2016, 2018, 2019*

### Basic Physics
- Speed of sound in tissue: c ≈ 1540 m/s (assumed constant)
- **B-scan (brightness mode):** pulse sent, echo time gives depth z = ct/2
- **Axial resolution:** ≈ c/(2f_centre) = half the pulse length; improves with higher frequency
- **Lateral resolution:** determined by beam width at focus; worsens with depth
- **Acoustic impedance:** Z = ρc = √(Ks·ρ); reflection at boundaries where Z changes

### Time-Gain Compensation (TGC)
Tissue attenuates ultrasound as it travels. Deeper echoes arrive later AND have less amplitude. TGC amplifies the received signal as a function of time after the pulse, increasing gain for later-arriving (deeper) echoes to compensate for attenuation. This gives uniform brightness across depths.

### Attenuation
```
Az = A₀ e^(−αz)   where α ∝ frequency
```
- Higher frequency → greater attenuation → less penetration depth
- Lower frequency → less attenuation → greater depth but lower resolution
- **Trade-off:** thyroid imaging at 8–15 MHz (shallow, high resolution); abdominal at 3–5 MHz (deep, lower resolution)

### Beamforming & Transmit Focus
For a linear array, delay outer elements so all wavefronts arrive simultaneously at the focal point:

```
Delay for element at lateral distance d from centre:
Δt = (√(z_focus² + d²) − z_focus) / c
```

Outer elements fire FIRST (before centre), so their signal has time to travel the extra distance.

### Dynamic Receive Focus
As the echo returns from increasing depth, continuously adjust receive delays so the array focuses on where the echo is coming from. This maintains lateral resolution at all depths.

**Delay formula as function of time t after transmission:**
```
d(t) = √(t²/4 + d²/c²) − t/2    (in seconds, where d = lateral offset of element)
```

### Sound Speed Errors (2018)
If real sound speed c_true ≠ c_assumed = 1540 m/s:
- **Axial distances:** measured as d_apparent = d_true × (c_assumed/c_true)
  - Fat (c < 1540): appears **too thick** (c_assumed/c_true > 1)
  - Liver (c > 1540): appears **too thin**
- **Lateral distances:** unaffected (determined by element spacing, not sound speed)
- **Triangle areas:** scale as one lateral × one axial → scale factor c_assumed/c_true. But in scan A vs scan B (same medium): areas are SAME since both use same assumed speed.

### Sample Q&A

**Q (2015):** 192-element probe, pitch 0.26 mm, 63 active elements, focus at 40 mm depth, c=1540 m/s. Calculate delay between outermost and centre elements.

**A:**
- Outermost element is (63−1)/2 = 31 elements from centre
- Lateral distance: d = 31 × 0.26 = 8.06 mm
- Distance from focal point to centre element: 40 mm
- Distance from focal point to outer element: √(40² + 8.06²) = √(1600 + 64.96) = √1664.96 = 40.804 mm
- Extra path for outer element: 40.804 − 40 = 0.804 mm
- Time delay: 0.804 mm / 1.54 mm/μs = **0.522 μs**
- Outer elements fire 0.522 μs BEFORE centre element.

**Q (2015):** Derive time at which outer element receives backscatter from depth z.

**A:** Pulse transmitted from centre at t₀. Round trip:
- Out: distance z (along centre-line) at speed c → arrives at focal point at t₀ + z/c
- Back: echo travels distance √(z² + 8.06²) to reach outer element → arrives at t₀ + z/c + √(z²+8.06²)/c = t₀ + (z + √(z²+8.06²))/c

In seconds with c = 1.54×10⁶ mm/s:
```
t = t₀ + (z + √(z² + 8.06²)) / (1.54 × 10⁶)
```

---

## 1E. Nuclear Medicine: PET & SPECT
*Appeared: 2017, 2019, 2024*

### SPECT vs PET

| Feature | SPECT | PET |
|---------|-------|-----|
| Full name | Single Photon Emission CT | Positron Emission Tomography |
| Radiation | Single γ-ray per decay | Two 511 keV γ-rays at 180° |
| Collimation | Mechanical (parallel-hole collimator) | Electronic coincidence detection |
| Sensitivity | Lower (collimator blocks most photons) | Higher (electronic collimation) |
| Resolution | Lower | Higher |
| Attenuation correction | Difficult (path varies with source depth) | Straightforward (CT measures full path) |
| Cost | Cheaper | More expensive |

### PET Physics
- Radionuclide emits positron (β⁺) → positron travels few mm → annihilates with electron
- Produces two 511 keV γ-rays at exactly 180° apart
- **Line of response (LOR):** connecting two simultaneously firing detectors
- **Coincidence window τ:** if both detectors fire within τ (~10 ns), event recorded

### Coincidence Timing Window
**Advantages of reducing τ:**
- Fewer random (accidental) coincidences → less background noise
- Better image quality

**Disadvantages of reducing τ:**
- May miss true coincidences from large patients (photon travel time across body ≈ 1 ns/30 cm)
- Requires faster detector electronics
- Current state-of-art: ~0.5 ns timing resolution possible, but standard τ ≈ 10 ns

### Attenuation Correction in PET
The measured coincidence count between detectors at d₁ and d₂ for source at a:
```
N(d₁,d₂) = λ(a) × exp[−∫_{d₁}^{d₂} μ(s) ds]
```
The attenuation factor depends only on the TOTAL path d₁→d₂, not on where along the path the source is. A CT scan measures μ(s) along all paths → allows correction of all LORs → enables filtered backprojection for λ(a).

### SNR in Nuclear Medicine
```
SNR ∝ √(number of detected counts)
    ∝ √(dose × measurement time)
```
- SNR vs time: √(time) — square root relationship
- SNR vs dose: √(dose) — square root relationship

### Scintillation Detector + Photomultiplier
1. High-energy γ-ray absorbed by NaI(Tl) crystal → excites crystal → emits light flash
2. Light hits **photocathode** → photoelectric effect → electrons emitted
3. Electrons accelerated through series of **dynodes** (each at higher voltage)
4. Each electron knocks out multiple secondary electrons → cascade amplification
5. Output pulse at anode: voltage proportional to incident γ-ray energy
6. **Energy discrimination:** reject Compton-scattered photons (lower energy) by threshold

### Four Collimation Types (2017)
1. **Gamma camera mechanical collimator:** parallel-hole lead plate; only photons perpendicular to detector pass through; determines direction of single-photon emitters
2. **PET electronic collimation:** coincidence detection between two detectors defines LOR; no mechanical collimator needed; much higher sensitivity
3. **3D PET septa (inter-ring shields):** removed in 3D mode for higher sensitivity but more scatter and randoms; 2D mode with septa: lower sensitivity, less scatter
4. **CT mechanical collimator:** pre-patient collimator shapes X-ray beam to slice thickness; post-patient collimator reduces scatter reaching detector

---

# PART 2: CURVES, SURFACES & INTERPOLATION

---

## 2A. Cubic Parametric Splines
*Appeared: 2010, 2013, 2014, 2016, 2017, 2021, 2023, 2024 — ALMOST EVERY YEAR*

### General Framework
```
p(t) = T M G
```
- **T** = [t³  t²  t  1] — parameter row vector
- **M** = 4×4 basis matrix (defines spline type)
- **G** = [p₀  p₁  p₂  p₃]ᵀ — control point geometry (each pᵢ can be 2D or 3D)
- t ∈ [0, 1] for one segment

**Spline surface:**
```
q(s,t) = S M Qᵢ Mᵀ Tᵀ
```
- S = [s³  s²  s  1], T = [t³  t²  t  1]
- Qᵢ = 4×4 matrix of i-coordinates of the 16 control points
- Separate evaluation for x, y, z coordinates

### Bezier Curves
```
M_Bezier = [−1   3  −3   1]
           [ 3  −6   3   0]
           [−3   3   0   0]
           [ 1   0   0   0]
```
**Properties:**
- Passes through p₀ at t=0 and p₃ at t=1
- Tangent at t=0: dp/dt = 3(p₁−p₀); tangent at t=1: dp/dt = 3(p₃−p₂)
- C₀ continuity at segment joins (share one endpoint)
- C₁ continuity if: p₁ of next segment is mirror of p₂ of current segment through p₃
- Convex hull property: entire curve lies within convex hull of {p₀,p₁,p₂,p₃}
- Can be subdivided into two Bezier segments

**Bezier basis functions (Bernstein polynomials):**
B₀(t) = (1−t)³, B₁(t) = 3t(1−t)², B₂(t) = 3t²(1−t), B₃(t) = t³

### B-Spline Curves
```
M_bspline = (1/6) [−1   3  −3   1]
                   [ 3  −6   3   0]
                   [−3   0   3   0]
                   [ 1   4   1   0]
```
**Properties:**
- Does NOT pass through any control point
- Start point (t=0): p(0) = (1/6)(p₀ + 4p₁ + p₂)  ← read last row of M × G
- End point (t=1): p(1) = (1/6)(p₁ + 4p₂ + p₃)
- Tangent at t=0: dp/dt = (1/2)(p₂ − p₀)
- C₂ continuity at segment joins (multi-segment shares 3 control points)
- Convex hull property: yes
- Best choice when smooth curve needed without strict positional requirements

**B-spline start point with repeated points (2023 key topic):**
- Normal: start at (p₀+4p₁+p₂)/6
- p₁ = p₂: start at (p₀+5p₁)/6 — one-sixth of way from p₁ to p₀
- p₀ = p₁ = p₂: start exactly at p₁ — useful for forcing endpoint
- p₁ = p₂ = p₃: end exactly at p₂ (analogously)

**Continuity with repeated points:**
- Normal join: C₂ continuity
- One repeated point at join: G₂ only
- Two repeated points at join: G₀ only (just position continuity, sharp corner possible)

### Catmull-Rom Curves
```
M_CR = (1/2) [−1   3  −3   1]
              [ 2  −5   4  −1]
              [−1   0   1   0]
              [ 0   2   0   0]
```
**Properties:**
- Passes through p₁ at t=0 and p₂ at t=1
- Tangent at p₁: (1/2)(p₂−p₀) — half the vector from previous to next control point
- Tangent at p₂: (1/2)(p₃−p₁)
- C₁ continuity at joins
- NO convex hull property
- Best for: smooth curve that must pass through specified points (camera paths, animation)

### Continuity Summary Table
| Spline | Passes through | Join C_n | Join G_n | Convex hull |
|--------|---------------|----------|----------|-------------|
| Bezier (1 shared pt) | p₀, p₃ | C₀ | G₁ possible | Yes |
| B-spline (3 shared pts) | Neither endpoint | C₂ | G₂ | Yes |
| Catmull-Rom (3 shared) | p₁, p₂ | C₁ | G₁ | No |

*C_n = parametric continuity (n-th derivative matches); G_n = geometric continuity (tangent direction matches but not magnitude)*

### Convex Hull Property
The curve lies entirely within the convex hull of its control points. Useful for: culling (if hull not in viewport, curve not drawn); checking for intersection with regions.

Catmull-Rom does NOT have this property → may need to convert to Bezier for efficient rendering.

### Closed Loops (2017)
For Catmull-Rom closed loop with n points p₁...pₙ:
- Each segment uses 4 consecutive points (with wraparound)
- Segment from pᵢ to p_{i+1} uses p_{i-1}, pᵢ, p_{i+1}, p_{i+2} (all mod n)
- The loop automatically has C₁ continuity everywhere

**Computing degenerate case (2017 a=0):** When a=0, all points on x-axis → collinear → curve degenerates to line segments along x-axis.

**Parametric vs geometric continuity at degeneracy:**
- Parametric velocity dp/dt may be zero at the degenerate point
- G₀ continuity (position): always maintained
- C₁, C₂ may fail if velocity → 0
- Need careful analysis: even if C₁ holds analytically, G₁ may fail if tangent vector is zero

### Bezier Subdivision
Matrix L subdivides a Bezier curve into two halves:
```
L = (1/8) [8  0  0  0]
           [4  4  0  0]
           [2  4  2  0]
           [1  3  3  1]
```
- New left segment control points: G_left = L × G
- Last row [1 3 3 1]/8 gives the midpoint of the original curve (t=0.5) = corner shared by both new segments

**Surface subdivision:** Apply L to rows of 4×4 control point grid → gives 4×7 grid → apply L^T to columns → gives 7×7 grid → forms 4 new Bezier patches.

**Sample Q (2024 style):** Two basis matrices M₁ and M₂ given. Which rows of G determine location/gradient at t=0?

**A:** At t=0, T = [0 0 0 1]. So p(0) = [0 0 0 1] M G = (last row of M) × G.
The gradient at t=0: dp/dt|_{t=0} = [0 0 1 0] M G = (third row of M) × G.

**Sample Q (2014 subdivision):** Find highest point on Bezier surface with z-values:
```
z = [0  1  1  0]
    [1  2  2  1]
    [1  2  2  1]
    [0  1  1  0]
```
By symmetry, maximum at centre (s=t=0.5). Apply last row of L = [1 3 3 1]/8 in both directions:
```
w = (1/64)[1  3  3  1]   ⊗   [1  3  3  1]
         = weight matrix w_ij = w_i × w_j / 8
```
z_max = (1/64)(1×0 + 3×1 + 3×1 + 1×0 + 3×1 + 9×2 + 9×2 + 3×1 + ...) = 96/64 = **1.5**

---

## 2B. Marching Cubes
*Appeared: 2010, 2017, 2019*

### Algorithm
1. Take 3D scalar volume (e.g., CT scan)
2. For each cubic cell of 8 voxels:
   - Classify each vertex as inside (≥ threshold) or outside (< threshold)
   - 2⁸ = 256 possible configurations → reduced to 15 by symmetry → lookup table
   - Look up triangle configuration for this case
   - Interpolate vertex positions along cell edges (where surface crosses)
3. Output: triangulated surface mesh

### Topological Problems
- **Ambiguous cases:** some configurations have two equally valid triangle connectivities → can produce holes or non-manifold geometry
- Without careful case handling, the mesh may not be watertight
- Solution: use extended lookup tables with 33 or more cases

### Surface Properties of Marching Cubes Output
- **Surface normal quality:** poor if computed from individual triangles; should use vertex normals averaged from volume gradient
- **Staircase artefacts:** at low resolution, facets are clearly visible; improves with higher resolution sampling
- **Isotropy errors (2017):** if sphere centre falls on grid point vs. off-grid, mesh looks different
  - On-grid: all edge intersections at same distance → perfect octahedral shape
  - Off-grid: different parts of sphere intersect at different edge positions → more irregular

**Width of Marching Cubes sphere (2017):**
- If f = 0.6² − (x−x₀)² − (y−y₀)² − (z−z₀)², threshold = 0
- Sphere has radius 0.6 voxels
- Case A (centre at grid point): all 6 edge midpoints at same distance → octahedral mesh, max width = 2 × (distance to nearest edge crossing) ≈ 0.72 voxels
- Case B (centre offset by 0.5 in x): asymmetric → different max width ≈ 1.11 voxels
- Case C (centre offset by 0.5 in x and y): → ≈ 0 voxels (all vertices outside or all inside)
- True width at high resolution: 2 × 0.6 = **1.2 voxels**

### Shape-Based Interpolation (2019 context)
Used to reconstruct 3D tumour volume from 2D CT cross-sections:
1. For each cross-section: threshold → binary image (inside/outside tumour)
2. Compute distance transform of each slice (positive inside, negative outside)
3. Linearly interpolate distance transforms between adjacent slices
4. Apply Marching Cubes to zero-isosurface of interpolated 3D distance volume

**Advantages over direct binary interpolation:** smooth surfaces, better topological properties, handles shape variations gracefully.

**Limitations:**
- Large slice spacing → shape may vary significantly between slices → linear interp inaccurate
- 2D distance transforms add margin horizontally but not vertically → use 3D distance transform to fix
- Acute angles between slice and tumour surface → 3D margin < 1 cm in some directions

---

## 2C. Laser Range Scanning
*Appeared: 2010, 2014, 2016 — less common recently*

### Triangulation Geometry
Setup: laser stripe projects vertical plane of light; camera at angle α observes where stripe hits surface.

**Distance formula (2016):**
```
d = L(f − x tan α) / (x + f tan α)
```
where: L = distance from camera focal point to laser plane (perpendicular), f = focal length, x = pixel offset from centre, α = camera angle from laser plane normal.

**Derivation:** From camera geometry: tan β = x/f. Laser plane is at distance L from camera focal point along normal. Using tan(α+β) = (tan α + tan β)/(1 − tan α tan β) and d = L/tan(α+β).

### Resolution
```
rₓ ≈ 2zθ / (1 + cos 2α)    (horizontal/depth resolution)
r_y ≈ 2zθ / sin 2α          (vertical resolution)
```
where θ = camera angular resolution per pixel. Both worsen linearly with distance z.

### Error Sources
1. **Surface specularity/translucency:** multiple reflections or light going through surface → wrong positions; fix: dust surface with white powder
2. **Camera pixel discretisation:** ±0.5 pixel uncertainty → depth error depends on dd/dx; worsens with distance
3. **Object movement:** object must stay still during scan (minutes); fix: rotate scanner instead of object
4. **Obscured features:** laser or camera line-of-sight blocked by surface features; fix: scan from multiple directions
5. **Finite laser thickness t:** edge of beam width causes position uncertainty t/tan θ; fix: narrower laser

### Sample Q (2016)
Given L=160 mm, f=12 mm, tan α=0.5, pixel width=2/3 mm, pixels at {0, 3, 9} from centre:

x = pixel × pixel_width = {0, 2, 6} mm

d = 160(12 − x×0.5)/(x + 12×0.5) = 160(12 − 0.5x)/(x + 6)

- x=0: d = 160×12/6 = **320 mm**
- x=2: d = 160×11/8 = **220 mm**
- x=6: d = 160×9/12 = **120 mm**

---

## 2D. Distance Transforms
*Appeared: 2015*

### Definition
A distance transform replaces each pixel with its shortest distance to the boundary of a predefined object.
- Positive values: inside object
- Negative values: outside object
- Zero: at boundary

### Types
| Type | Directions | Far-field shape | Notes |
|------|-----------|-----------------|-------|
| City-block | 4 (H+V only) | Diamond | Fastest |
| Chamfer | 8 (H+V+diagonal) | Octagon | Good approximation |
| Euclidean | All directions | Circle | Most accurate, most complex |

### Two-Pass Algorithm
1. Forward pass (top-left to bottom-right): use forward mask, replace each pixel with minimum of (mask value + neighbour value)
2. Backward pass (bottom-right to top-left): use backward mask

### Applications
1. **Shape-based interpolation:** create intermediate shapes between two 2D cross-sections; linearly interpolate distance transforms; threshold at zero
2. **Shape expansion/erosion:** threshold at ±r to dilate or erode boundary by r pixels; used in radiotherapy planning (add margin to tumour)
3. **Skeletonisation:** find ridge/peak of positive distance transform → skeleton of shape
4. **Registration:** align shapes by minimising distance transform value differences

### Far-field Behaviour
At large distances from a complex object, the far-field shape is determined by the distance transform TYPE, not the object shape. City-block → diamond; chamfer → octagon; Euclidean → circle. Small features of the object have limited local effect on the transform.

---

## 2E. Interpolation Methods
*Appeared: 2018, 2021, 2024 — growing trend*

### Nearest Neighbour
- Each query point assigned the value of the closest sample
- Produces step discontinuities (staircase artefacts)
- Very fast; appropriate when speed > quality
- iso-contour at value V: Voronoi boundary between regions with value <V and ≥V

### Bilinear Interpolation (2D)
For 4 surrounding samples at corners (0,0), (1,0), (0,1), (1,1) with values f₀₀, f₁₀, f₀₁, f₁₁:
```
f(α,β) = (1−α)(1−β)f₀₀ + α(1−β)f₁₀ + (1−α)βf₀₁ + αβf₁₁
```
- C⁰ continuous (values match at boundaries, gradients may not)
- Iso-contour: hyperbolic curves (not straight lines)
- For unit square with values (0,0,0,2): iso-contour at f=1 is y = 1 − 1/(2(1−x)) — hyperbola

### Delaunay Triangulation
**Definition:** Connect points so that no point lies inside the circumcircle of any triangle.

**Properties:**
- Maximises minimum angle (avoids very thin triangles)
- For 4 points forming a convex quadrilateral: two possible diagonals → two valid triangulations
- For 4 points on a common circumcircle: both triangulations are equally valid Delaunay (degenerate case)
- Within each triangle: linear (barycentric) interpolation

**Iso-contour:** piecewise linear within each triangle; interpolate linearly along each edge and connect.

### Radial Basis Functions (RBF)
```
s(x) = c₀ + c₁x + c₂y + ... + Σᵢ₌₁ᴺ λᵢ φ(||x − xᵢ||)
```
- φ(r) = basis function (only depends on distance r from data point)
- λᵢ = weight for data point i
- Polynomial terms for well-posedness

**Multiquadric basis (2021):** φ(r) = √(r² + α²)
- α = 0: reduces to φ(r) = r (piecewise linear at data points, discontinuous gradient)
- Large α: smoother, less interpolation (data points not exactly reproduced)
- Sensible α ≈ typical spacing between data points

**Setting up the system:** For N data points, solve N linear equations:
```
s(xᵢ) = fᵢ   for all i
```
Plus orthogonality conditions Σ λᵢ = 0, Σ λᵢ xᵢ = 0, etc. (ensures polynomial terms are unique)

Result is a linear system in [λ₁...λₙ, c₀, c₁, ...].

**Sample Q (2021):** Three data points x={0,1,5}, f={1,4,3}, multiquadric with α=0.6.
Matrix equation has entries φᵢⱼ = √((xᵢ−xⱼ)²+α²):
```
φ₁₁ = √(0+0.36) = 0.6
φ₁₂ = √(1+0.36) = 1.166
φ₁₃ = √(25+0.36) = 5.036
etc.
```
Solve for λᵢ and c₀, c₁. Then evaluate s(x) at any x.

---

# PART 3: 3D GRAPHICAL RENDERING

---

## 3A. Phong Reflection Model
*Appeared: 2010, 2014, 2015, 2017, 2018, 2021, 2023 — ALWAYS IN EXAM*

### The Formula
**Monochromatic Phong:**
```
I = Iₐkₐ + Ipkd(L·N) + Ipks(R·V)ⁿ
```

**Full colour with shadow and attenuation:**
```
I_λ = c_λ Iₐkₐ + Σᵢ Sᵢ f_att Ipᵢ [c_λkd(Lᵢ·N) + ks(Rᵢ·V)ⁿ]
```

### Complete Term Glossary
| Symbol | Meaning |
|--------|---------|
| I_λ | Intensity of reflected light for colour λ ∈ {r,g,b} |
| c_λ | Surface colour (0 = no colour, 1 = full colour); e.g., cᵣ=1, c_g=0, c_b=0 for red |
| Iₐ | Ambient (background) light intensity |
| kₐ | Ambient reflection coefficient (0=absorbs all ambient, 1=reflects all) |
| Ip | Point light source intensity |
| kd | Diffuse reflection coefficient (0=dark/absorbing, 1=bright) |
| L | Unit vector from surface point TOWARDS light source |
| N | Unit outward surface normal at the point |
| ks | Specular reflection coefficient (0=matte, 1=mirror-like) |
| R | Mirror reflection of L about N: R = 2(L·N)N − L |
| V | Unit vector from surface point TOWARDS viewer |
| n | Specular exponent: high n = tight highlight; low n = broad highlight |
| Sᵢ | Shadow factor: 0=fully shadowed, 1=fully lit |
| f_att | Distance attenuation: min(1/(a₁+a₂d+a₃d²), 1) |

**Note:** L·N must be clamped to 0 (no illumination from behind). Similarly R·V clamped to 0.

### Mirror Reflection Vector
```
R = 2(L·N)N − L
```
**Derivation:** R is the reflection of L in the plane with normal N. Project L onto N: (L·N)N. The component along the surface plane: L − (L·N)N. Reflect in the plane: negate surface component. R = (L·N)N − [L − (L·N)N] = 2(L·N)N − L.

### Blinn's Approximation (Halfway Vector)
```
H = (L + V) / |L + V|    (halfway between L and V)
I'_λ = c_λ(Iₐkₐ + Ipkd(L·N)) + Ipks(N·H)ⁿ
```
**Why use Blinn:**
- Avoids computing R (single dot product N·H instead of vector calculation + R·V)
- If L and V are at infinity (distant light and viewer), H is CONSTANT across entire scene → very efficient
- Produces very similar specular highlights to original Phong (minor difference in glint shape)

### Unnormalised Normals (2010, 2021)
If N is NOT renormalised after interpolation:
- **Diffuse term kd(L·N):** scales as |N| → too bright where |N| > 1 (convex surfaces after interpolation), too dark where |N| < 1; approximately acceptable for diffuse-only rendering
- **Specular term ks(R·V)ⁿ:** R = 2(L·N)N − L depends on |N|; R is no longer unit vector; glint intensity distorted nonlinearly → **unacceptable** for specular

**Conclusion:** Acceptable for purely diffuse; NOT acceptable for specular highlights.

### Sample Q&A

**Q:** Explain all terms in I_λ = c_λ(Iₐkₐ + Ipkd**L·N**) + Ipks(**R·V**)ⁿ.

**A:** I_λ is the total reflected intensity of colour λ. The first term c_λIₐkₐ is the ambient contribution: c_λ specifies the surface colour (fraction of each wavelength reflected), Iₐ is background illumination intensity, kₐ is the fraction of ambient light reflected. The second term c_λIpkd(**L·N**) is diffuse: Ip is point light intensity, kd is the surface's diffuse reflectivity, **L·N** is the cosine of the angle between the light direction and surface normal (Lambert's law — maximum when facing light, zero at glancing angle). The third term Ipks(**R·V**)ⁿ is specular: ks is specular reflectivity, **R** is the mirror reflection of L about N, **V** is the direction to the viewer, and **R·V** is the cosine of the angle between the mirror and viewing directions — maximum when viewer is looking along the mirror direction. The exponent n controls highlight tightness: large n = mirror-like, small n = broad matte highlight.

---

## 3B. Gouraud vs Phong Shading
*Appeared: every year alongside Phong model*

### Gouraud Shading
1. Compute vertex normals (average of surrounding triangle normals)
2. Apply Phong reflection model at each VERTEX → get colour/intensity per vertex
3. Rasterise triangle: **bilinearly interpolate colours** across pixels
4. Very efficient: per-pixel work is just interpolation (can use incremental computation)

**Weaknesses:**
- Specular highlights missed if highlight falls between vertices (most pixels show no specular)
- Even if specular computed at vertices, interpolated colours look wrong between vertices
- Mach banding: human eye sensitive to gradient changes → sees polygon boundaries
- Polygon edges obvious near sharp highlights

### Phong Shading
1. Compute vertex normals
2. Rasterise triangle: **bilinearly interpolate surface NORMALS** across pixels
3. Renormalise interpolated normal at each pixel (|N_interp| ≠ 1)
4. Apply full Phong reflection model at each PIXEL

**Strengths:**
- Accurately reproduces specular highlights anywhere on surface
- Better simulation of surface curvature
- No edge artefacts near highlights

**Weaknesses:**
- More expensive (full lighting calculation per pixel)
- Must renormalise normal at each pixel

### Why Gouraud Shows Polygon Edges Near Highlights
The specular term (R·V)ⁿ is highly nonlinear. Linear interpolation of intensity across a triangle cannot reproduce this nonlinearity. At the boundary of one polygon, the colour jumps discontinuously to the next polygon's interpolated colours. The human visual system is especially sensitive to intensity gradient changes (Mach banding effect), making polygon boundaries highly visible near specular regions.

### Hardware Phong Shading (Pixel Shaders)
**Minimum rasteriser requirement:** 4 scalar values per pixel: Nₓ, Ny, Nz (normal components) + z (depth).

**In pixel shader:**
1. Receive interpolated Nₓ, Ny, Nz
2. Renormalise: N = [Nₓ, Ny, Nz]/|[Nₓ, Ny, Nz]|
3. Compute I_λ = c_λ(Iₐkₐ + Ipkd(L·N)) + Ipks(N·H)ⁿ
   - L is constant (assuming distant light)
   - H = (L+V)/|L+V| is constant if viewer at infinity too

**GLSL example (2021):**
```glsl
spec = pow(max(dot(V, R), 0.0), n);
```
- `n`: specular exponent; controls tightness of highlight
- This goes in **fragment shader** (not vertex shader) because we need per-pixel normals for Phong shading; in vertex shader only vertex values are available → that would be Gouraud

### Vertex Normal Computation (2023)
For central vertex o surrounded by vertices v₀...v₅:
```
nᵥ = (1/2) Σᵢ vᵢ × v_{i⊕1}
```
where ⊕ is addition modulo n (circular). This formula is independent of o.

**Physical meaning:** nᵥ represents the area of the polygon defined by v₀...v₅ projected onto the plane perpendicular to nᵥ. It's the summed surface normal weighted by triangle area.

**Limitation:** Formula gives incorrect normal if central vertex is far from the plane of surrounding vertices, or if surrounding triangles vary greatly in size.

---

## 3C. Z-Buffer & Hidden Surface Removal
*Appeared: 2015, 2016, 2019, 2024*

### Z-Buffer Algorithm
1. Initialise z-buffer to 1.0 (maximum depth, at far plane)
2. For each polygon → rasterise to pixels
3. For each pixel (x,y): compute screen-space depth zₛ by interpolation
4. If zₛ < z-buffer[x,y]:
   - Write colour to frame buffer
   - Update z-buffer[x,y] = zₛ
5. Final frame buffer shows nearest surface at each pixel

### Depth Transform
View coordinate zᵥ (negative, −f to −n) → screen coordinate zₛ (0 to 1):
```
zₛ = f(1 + n/zᵥ) / (f − n)
```
Or equivalently:
```
zₛ = (f×zᵥ + f×n) / (zᵥ(f−n)) = -(f+n)/(f-n) - 2fn/((f-n)zᵥ)  (OpenGL form differs by sign convention)
```
**Near plane (zᵥ = −n):** zₛ = 0
**Far plane (zᵥ = −f):** zₛ = 1

### Why Nonlinear?
The nonlinear relationship is necessary to **preserve collinearity**: straight lines in 3D view space must remain straight lines in 3D screen space. This ensures that linear interpolation of zₛ across triangles gives correct values at interior pixels. The nonlinearity does NOT cause perspective foreshortening (that comes from the perspective divide w = −zᵥ).

**Common wrong answer:** "To make far plane objects smaller" — WRONG. Perspective foreshortening comes from dividing xᵥ, yᵥ by −zᵥ, not from the z mapping.

### Precision Analysis
For a k-bit integer z-buffer, the quantisation step in zₛ is 2^(−k). The minimum resolvable depth difference δ at view-space depth zᵥ:

Working from first principles: at the far plane zᵥ = −f + δ, the last quantisation level is 1 − 2^(−k):
```
1 − 2^(−k) = f(1 + n/(−f+δ))/(f−n) = 1 − nδ/(f(f−n)/(f)) ≈ 1 − nδ/f²
```
Therefore:
```
2^(−k) = nδ/f²
k = −log₂(nδ/f²)
```

**Standard result: k ≈ 24 bits** for typical settings (n=1m, f=1000m, δ≈few mm)

**Effect of n and f:**
- Large f/n ratio → precision concentrated near near plane, very poor at far plane
- Increasing n: improves overall precision (2^(−k) = nδ/f² → same δ requires larger n if k fixed → smaller δ resolvable)
- Setting n tighter (closer to nearest object): dramatically improves far-plane precision

**Z-fighting:** Two nearly-coplanar surfaces map to same zₛ → arbitrary which gets drawn. Fix: increase n, or use polygon offset extension.

### Floating-Point Z-Buffer (2024)
Compared to k-bit integer:
- **Advantages:** More precision near camera (where needed most); no fixed quantisation step → smooth transition everywhere; avoids z-fighting at near plane
- **Disadvantages:** Floating-point comparison is slower than integer; k bits of floating point ≠ k bits of integer precision (some bits used for exponent); may not improve far-plane precision
- **Key insight:** Integer z-buffer puts most bits near the far plane (nonlinear zₛ mapping). Floating-point naturally puts more bits near zero (near plane) → better match to visual importance.

### Sample Q&A (2015 Z-buffer precision)
Wide corridor, walls 200m apart, light panelling 6mm in front of walls. n=1m, f=1000m. Artefact (wall showing through panelling) visible at far end.

**Q:** Estimate z-buffer precision in bits.

**A:** At the far plane, the wall is at zᵥ = −1000+0 = −1000 m. The panelling is at zᵥ = −1000+0.006 = −999.994 m. The δ = 0.006 m must be unresolvable. Using k = −log₂(nδ/f²) = −log₂(1×0.006/10⁶) = −log₂(6×10⁻⁹) = log₂(1.67×10⁸) ≈ 24.0 bits.

**Answer: 24 bits**

**Why doesn't artefact affect visible outer edge?** At the near part of the corridor (zᵥ ≈ −n), precision is very high → the 6mm difference is easily resolved. The artefact only occurs near the far plane where precision is lowest.

---

## 3D. Shadow Z-Buffer
*Appeared: 2023*

### Algorithm
1. Render scene from **light's viewpoint** → store depth values in **shadow z-buffer** (zₛ')
2. For each pixel in the **camera view**:
   a. Find world-space position of surface point
   b. Transform to light's clip space → get z' (depth from light's POV) and zb (stored shadow buffer value)
   c. If z' > zb + ε: point is in shadow (something is closer to the light)
   d. If z' ≤ zb + ε: point is lit

### Artefacts

**Peter Panning:**
- Shadow detaches from base of walls/objects → "object floating"
- Cause: bias ε too large → shadow pushed forward beyond actual contact point
- Fix: reduce ε, or use slope-scaled depth bias

**Shadow Acne (Self-Shadowing):**
- Moire pattern of self-shadowing across lit surface
- Cause: a surface maps to the same shadow z-buffer pixel as itself, but due to finite spatial resolution of shadow buffer and oblique viewing angle, z' > zb (appears to be in own shadow)
- NOT the same as bit-depth precision issue — it's a **spatial resolution** issue
- Worse for oblique illumination and distant light sources (large perspective compression)
- Fix: add bias ε; use slope-scaled bias: ε proportional to angle between surface and light rays (larger ε for more oblique surfaces)

**Slope-Scaled Depth Bias:**
```
ε = constant + slope_scale × max(|∂z/∂x|, |∂z/∂y|)
```
This adds minimal bias where surface faces light directly, more bias for oblique surfaces.

---

## 3E. View Volumes & Projection Matrices
*Appeared: 2016, 2019*

### Projection Matrix
Maps view coordinates (xᵥ, yᵥ, zᵥ) to homogeneous screen coordinates:
```
[w·xₛ]   [d/xmax     0          0           0    ] [xᵥ]
[w·yₛ] = [  0      d/ymax       0           0    ] [yᵥ]
[w·zₛ]   [  0        0      −f/(f−n)  −fn/(f−n) ] [zᵥ]
[ w  ]   [  0        0         −1           0    ] [ 1]
```

After homogeneous divide by w = −zᵥ: xₛ = xᵥ·d/(−zᵥ·xmax), yₛ = yᵥ·d/(−zᵥ·ymax), zₛ as before.

### Reading Parameters From Matrix
Given concrete matrix:
```
[3/4   0     0          0   ]
[ 0    1     0          0   ]
[ 0    0   −100/99  −1000/99]
[ 0    0    −1          0   ]
```
- **Field of view (y):** ymax/d = 1/(first element if equal) → 2tan⁻¹(1) = 90°
- **Field of view (x):** xmax = 1, d = 3/4, so 2tan⁻¹(xmax/d) = 2tan⁻¹(4/3) = **106.3°**
- **Near plane n:** from −fn/(f−n) ÷ (−f/(f−n)) = n → (1000/99)/(100/99) = 10 → **n = 10**
- **Far plane f:** from −f/(f−n) = −100/99 and n=10 → f = 100n/99 × 99/1 ... solving: **f = 1000**

### Field of View
```
FOV_y = 2 tan⁻¹(ymax/d)
FOV_x = 2 tan⁻¹(xmax/d)
```
Aspect ratio: xmax:ymax. If window has different aspect ratio → distortion.
- 640×480 window with 4:3 projection matrix → no distortion
- 1920×1080 (16:9) with 4:3 projection → objects stretched horizontally by 4/3 × 9/16 = 3/4... wait, stretched by (window_aspect)/(projection_aspect) = (16/9)/(4/3) = 4/3.

### Off-Screen Picking Projection (2016)
To select a triangle by clicking a pixel, render to narrow view volume around click point:
- Reduce xmax and ymax so 1 pixel = entire view volume
- For 640×480 window, click at centre: new xmax = xmax_original/(640/2) = xmax/320; new ymax = ymax_original/(480/2) = ymax/240
- Scale x factor 3/4 by 320 → 240; scale y factor 1 by 240 → 240

**Off-screen matrix:**
```
[240    0      0          0   ]
[  0  240      0          0   ]
[  0    0   −100/99  −1000/99]
[  0    0    −1          0   ]
```

**If click not at centre:** view volume is not symmetric → off-centre perspective. More complex projection matrix needed (asymmetric frustum), which the simple 3G4 formulation cannot represent. OpenGL supports this with glFrustum but the matrix form has additional non-zero entries.

**Selecting from multiple surviving triangles:** use depth test — choose the triangle with the smallest zₛ value (closest to viewer), which is the visible one the user intends to select.

### Sutherland-Hodgman Clipping (2019)
Clip polygon against each boundary of view volume in turn. In homogeneous screen coordinates:
- Right boundary: Xₛ = w (i.e., xₛ ≤ 1 → Xₛ ≤ w)
- Left boundary: Xₛ = −w
- Etc.

For each edge crossing a boundary, compute intersection point by linear interpolation in homogeneous coordinates.

---

## 3F. Ray Tracing
*Appeared: 2016 — rare recently*

### Recursive Ray Tracing
1. Cast ray V from viewpoint through each pixel
2. Find first intersection with scene objects
3. At intersection point P₁, spawn three types of rays:
   - **Shadow rays Lᵢ:** to each light source; if blocked → shadow (Sᵢ = 0)
   - **Reflection ray R₁:** in mirror direction; recurse to find I_r
   - **Refraction ray T₁:** through surface using Snell's law; recurse to find I_t
4. Pixel intensity:
```
I_λ = c_λIₐkₐ + Σᵢ Sᵢ f_att Ipᵢ(c_λkd(Lᵢ·N) + ks(Rᵢ·V)ⁿ) + ks·I_r + kt·I_t
```
5. Recursion continues until maximum depth d or no intersection

### Intersection Test Count (Naive)
Per pixel: 1 primary ray tests nₚ polygons. At each intersection: nₗ shadow rays + 1 reflection + 1 refraction = nₗ+2 rays → each tests nₚ polygons.

At depth d, the tree has 2⁰+2¹+...+2^d = 2^(d+1)−1 nodes (reflection/refraction branches).
Each node also spawns nₗ shadow rays.

Total intersection tests:
```
nᵣ × nₚ × (1 + nₗ) × (2^(d+1) − 1)
```

### Acceleration: Voxel Grid (3D-DDA)
```
while not finished:
  output current voxel (X, Y, Z)
  if next_x < next_y and next_x < next_z:
    next_x += dx; X += sign(ray_x)
  elif next_y < next_z:
    next_y += dy; Y += sign(ray_y)
  else:
    next_z += dz; Z += sign(ray_z)
```
Only check polygons associated with each visited voxel.

**Teapot in stadium problem:** Most voxels are empty (stadium). Teapot fits in a few voxels. Performance fine. But if voxel size covers whole stadium, teapot in one voxel → check all teapot polygons per ray. Solution: **non-uniform grid (k-d tree, octree)** — fine subdivision around teapot, coarse for empty stadium.

---

## 3G. Texture Mapping
*Appeared: 2018*

### Basics
- Each vertex has texture coordinates (u,v) ∈ [0,1]²
- Look up texture image at interpolated (u,v) for each pixel
- Must use perspective-correct interpolation

### Perspective-Correct Interpolation
Simple linear interpolation of (u,v) is WRONG in perspective — gives affine warping.

Correct formula: interpolate u/z and 1/z linearly, then divide:
```
u_α = [(1−α)(u₀/z₀) + α(u₁/z₁)] / [(1−α)(1/z₀) + α(1/z₁)]
```

**Behaviour:**
- z₁ = z₀: linear (correct — no perspective distortion)
- |z₁| > |z₀| (far end further): texture compresses near far end (more texture per pixel near far)
- |z₁| >> |z₀|: extreme compression at far end

### Texture Artefacts & Solutions

**Magnification (texel larger than pixel — near camera):**
- Artefact: blocking/pixellation (nearest-neighbour) or blurring (bilinear)
- Fix: bilinear texture filtering (interpolate between 4 nearest texels)

**Minification (texel smaller than pixel — far from camera):**
- Artefact: aliasing, shimmering
- Cause: undersampling of texture → aliasing
- Fix: **Mipmapping** — pre-compute downsampled versions at 1/2, 1/4, 1/8 resolution; select appropriate mip level based on pixel footprint size; optional trilinear filtering between mip levels

---

## 3H. Triangle Meshes & Vertex Normals
*Appeared: 2014, 2017, 2023*

### Storage Representations
**Option A: Single triangle list**
- Store all three vertex positions for each triangle
- Memory: 3 vertices × 3 floats × N_triangles
- Vertex shared by 6 triangles → stored 6 times
- Problem: same vertex stored multiple times → rounding errors, inconsistent positions

**Option B: Vertex list + Triangle index list**
- Vertex list: (x,y,z) per unique vertex
- Triangle list: (i,j,k) indices into vertex list
- Memory: less (each vertex stored once)
- Benefits: floating-point consistency, easier editing (move vertex by changing one entry), consistency checking (e.g., watertight mesh)

### Triangle Normal
Given triangle with vertices A, B, C:
```
n = (B−A) × (C−A)
n̂ = n / |n|    (unit normal)
```
Direction follows right-hand rule: counterclockwise winding → normal points outward (towards viewer in OpenGL convention).

**Area of triangle:** A = |n|/2 = |(B−A)×(C−A)|/2

### Polygon Area Formula (2014)
For polygon with N vertices v₁...vₙ:
```
p = (1/2) Σᵢ₌₁ᴺ vᵢ × v_{i⊕1}
```
where ⊕ = addition modulo N.

- |p| = area of polygon (projected onto plane perpendicular to p)
- Direction of p = polygon normal

**Verification for triangle (A,B,C):**
```
p = (1/2)(A×B + B×C + C×A) = (1/2)(B−A)×(C−A)  ✓
```

### Vertex Normal (2023)
For a vertex o surrounded by vertices v₀...v₅ (forming 6 triangles with o):
```
nᵥ = (1/2) Σᵢ₌₀⁵ vᵢ × v_{i⊕1}
```
**Key result:** This does NOT depend on o at all. It equals the area and normal of the polygon formed by v₀...v₅.

**Physical meaning:** |nᵥ| = area of surrounding polygon projected onto plane with normal nᵥ. This is the largest projected area of the 3D polygon, giving an estimate of total polygon area.

**When approximation is poor:**
- Surrounding triangles are very different sizes → weighted towards larger triangles
- Central vertex is far out of plane of surrounding vertices (e.g., on a spike) → surrounding polygon normal doesn't represent actual vertex normal well
- Works best when triangles are similar size and central vertex is close to average plane of neighbours

**Rendering normals:** Must be defined at vertices (not per triangle) to allow interpolation across mesh during Gouraud or Phong shading.

### Surface Rendering Pipeline (2010)
**Block diagram:**
```
Object coords → [Model transform] → World coords
→ [View transform] → View coords
→ [Projection + Clip] → Homogeneous screen coords
→ [Perspective divide] → Normalised screen coords
→ [Viewport transform] → Pixel coords
→ [Rasterise + Z-test + Shade] → Frame buffer
```

### Back-Face Culling
Discard triangles whose outward normal points away from viewer (N·V < 0). Only valid for:
- Closed opaque objects (inside faces never visible)
- Single-sided surfaces
**NOT valid for:** transparent objects, open surfaces, objects where camera can be inside.

Approximately halves the number of triangles to rasterise.

---

# PART 4: PATTERN ANALYSIS & 2026 PREDICTIONS

---

## Topic Frequency Table (2010–2024)

| Topic | Years | Frequency | Priority |
|-------|-------|-----------|----------|
| Phong model + Gouraud/Phong shading | 2010,2014,2015,2017,2018,2021,2023 | 7/11 | **MUST KNOW** |
| Splines (Bezier/B-spline/Catmull-Rom) | 2010,2013,2014,2016,2017,2021,2023,2024 | 8/11 | **MUST KNOW** |
| Z-buffer depth/precision | 2015,2016,2019,2024 | 4/11 | **HIGH** |
| Radon Transform/Sinogram | 2010,2013,2014,2016,2018 | 5/11 | **HIGH** |
| MRI | 2010,2015,2019,2023 | 4/11 | **HIGH** |
| Ultrasound | 2015,2016,2018,2019 | 4/11 | **HIGH** |
| Nuclear Medicine (PET/SPECT) | 2017,2019,2024 | 3/11 | MEDIUM-HIGH |
| Interpolation (NN/bilinear/Delaunay/RBF) | 2018,2021,2024 | 3/11 | MEDIUM-HIGH |
| Marching Cubes | 2010,2017,2019 | 3/11 | MEDIUM |
| Laser Range Scanning | 2010,2014,2016 | 3/11 | LOW (fading) |
| Triangle mesh/vertex normals | 2014,2017,2023 | 3/11 | MEDIUM |
| AART/CT reconstruction | 2010,2021 | 2/11 | MEDIUM |
| View projection matrices | 2016,2019 | 2/11 | MEDIUM |
| Shadow z-buffer | 2023 | 1/11 | MEDIUM |
| Texture mapping | 2018 | 1/11 | LOW |
| Ray Tracing | 2016 | 1/11 | LOW |
| Distance Transforms | 2015 | 1/11 | LOW |

---

## Year-by-Year Topic Map

| Year | Q1 | Q2 | Q3 | Q4 |
|------|----|----|----|----|
| **2010** | AART (CT) | Radon+MRI | Splines (B-spline+CR) | Marching Cubes+Laser / Rendering pipeline+Phong |
| **2012** | (unavailable) | | | |
| **2013** | Radon Transform | B-spline computations | — | Phong model |
| **2014** | Radon+Sinogram | Laser Scan+Polygon area | Splines+Subdivision | Phong+Gouraud+Hardware |
| **2015** | Ultrasound focus | MRI (T₁,T₂,spin-echo) | Distance Transforms | Z-buffer precision |
| **2016** | Ultrasound+Radon | Laser+B-spline | Projection matrices+FOV | Ray Tracing+Voxels |
| **2017** | Collimation+MRI localisation | Marching Cubes+Normals | Catmull-Rom+closed loop | Phong+Gouraud artefacts |
| **2018** | Ultrasound speed errors | Radon+Sinogram | 2D Interpolation (bilinear,Delaunay,RBF) | Texture Mapping+Phong |
| **2019** | Attenuation (US,X-ray,PET) | MRI (T₁,T₂,FID) | Shape-based interpolation | View volumes+Clipping |
| **2021** | AART+Iterative CT | Splines (CR+BS, closed) | Radial Basis Functions | Phong+Blinn+Specular |
| **2023** | MRI+Slice selection+k-space | Triangle meshes+normals | B-spline+repeated points | Phong+Shadow z-buffer |
| **2024** | Nuclear Med/PET/SPECT | Interpolation (NN,bilinear,Delaunay) | Splines+Subdivision | Z-buffer float vs int |

---

## Trends and Patterns

### Consistent Patterns
1. **Phong shading appears EVERY year** — no exception since records began
2. **Splines appear every year or nearly so** — 8 out of 11 years
3. **One medical imaging topic per paper** — always
4. **Usually one "computation" question** mixing medical/curves (e.g., Radon calculation, spline matrix computation, AART worked example)

### Rotating Medical Topics
- **MRI:** 2010, 2015, 2019, 2023 → appears roughly every 4 years; **2027 would be next by cycle**
- **Ultrasound:** 2015, 2016, 2018, 2019 → was common, absent since 2019 → **overdue**
- **Nuclear medicine:** 2017, 2019, 2024 → increasing frequency → **high probability**
- **Radon/CT:** 2010, 2013, 2014, 2016, 2018 → absent since 2018 → **overdue**

### Recent Trends (2021–2024 = likely to continue)
- **Floating-point precision** analysis (z-buffer, interpolation accuracy)
- **Theoretical continuity analysis** (spline joins, repeated points, G₀/G₁/G₂/C₁/C₂)
- **Nuclear medicine** questions growing
- **Interpolation theory** (RBF, Delaunay) replacing geometric topics like laser scanning

### Fading Topics (not seen since 2016)
- Ray tracing, laser scanning, distance transforms, view projection matrices — unlikely to return but possible if syllabus unchanged

---

## 2026 Exam Predictions

### Almost Certain (bet your grade on these)
| Topic | Why | Key subtopics to nail |
|-------|-----|----------------------|
| **Phong + Gouraud/Phong shading** | Every single year | Full formula, all terms, Blinn's H, vertex normals, hardware pixel shader, unnormalised normals |
| **Splines** (Bezier/B-spline/CR) | 8/11 years | All 3 basis matrices, continuity at joins, convex hull, subdivision matrix L, surface formula, repeated points |

### Very Likely
| Topic | Why | Key subtopics |
|-------|-----|---------------|
| **Ultrasound** | Absent since 2019, overdue | Beamforming delays, TGC, attenuation, sound speed errors, dynamic focus |
| **Radon Transform** | Absent since 2018, overdue | Definition, strip/disk/point formulas, sinogram, two strips same wv |
| **Z-buffer + floating point** | 2024 was float vs int, likely follow-up | Depth formula, nonlinearity reason, precision calculation k bits |

### Likely
| Topic | Why | Key subtopics |
|-------|-----|---------------|
| **Nuclear Medicine / PET** | Appeared 2024, strong trend | PET vs SPECT, coincidence window, attenuation correction, SNR |
| **MRI** | 2023 introduced k-space, topic expanding | Slice selection, phase/frequency encoding, k-space, T₁/T₂ weighting |
| **Interpolation** | 2021 and 2024 had it | Bilinear, Delaunay, RBF with multiquadric, role of α |

### Possible (know the basics)
| Topic | Why | Minimum to know |
|-------|-----|-----------------|
| **AART/CT reconstruction** | 2021 was last time; 4-5 year cycle | Algorithm, relaxation factor, optimal ordering, polychromatic problem |
| **Marching Cubes** | Not since 2019 | Algorithm, 256 cases, topological problems, normals |
| **Triangle meshes** | 2023 was recent | Storage schemes, vertex normal formula, polygon area |
| **Shadow z-buffer** | New in 2023, could continue | Algorithm, Peter Panning, shadow acne, slope-scaled bias |

---

# APPENDIX: KEY FORMULAS QUICK REFERENCE

## Medical Imaging
```
Beer-Lambert:          I = I₀ e^(−μx)
AART update:           μᵢ_new = μᵢ_old + λ(Q_meas − Q_est)/N_pixels
Radon of strip(w,v):   R = wv/sinθ
Radon of disk(r):      R = 2√(r²−s²) for |s|≤r
Point source sinogram: s = a cosφ + b sinφ
Larmor frequency:      f = γB₀  (γ = 42.58 MHz/T for protons)
T₁ recovery:           Mz = M₀(1 − e^(−t/T₁))
T₂ decay:              Mxy = M₀ e^(−t/T₂)
Ultrasound delay:      Δt = (√(z² + d²) − z)/c
US attenuation:        Az = A₀ e^(−αz)
PET attenuation:       N(d₁,d₂) = λ(a) exp[−∫ μ(s)ds]
SNR:                   ∝ √(dose × time)
```

## Splines
```
General:               p(t) = T M G,  T = [t³ t² t 1]
B-spline start:        p(0) = (p₀+4p₁+p₂)/6
B-spline end:          p(1) = (p₁+4p₂+p₃)/6
CR passes through:     p₁ at t=0, p₂ at t=1
Subdivision midpoint:  p(0.5) = (p₀+3p₁+3p₂+p₃)/8  (last row of L, Bezier)
Surface:               q(s,t) = S M Qᵢ Mᵀ Tᵀ
```

## Rendering
```
Phong:     I_λ = c_λ(Iₐkₐ + Ipkd(L·N)) + Ipks(R·V)ⁿ
Mirror:    R = 2(L·N)N − L
Blinn:     H = (L+V)/|L+V|;  replace (R·V)ⁿ with (N·H)ⁿ
Z-buffer:  zₛ = f(1 + n/zᵥ)/(f−n)
Precision: k = −log₂(nδ/f²)  (k bits, δ = min resolvable depth diff)
FOV:       2tan⁻¹(xmax/d)
Polygon:   p = (1/2)Σᵢ vᵢ × v_{i⊕1}
Vertex N:  nᵥ = (1/2)Σᵢ vᵢ × v_{i⊕1}  (independent of centre)
Ray tests: nᵣ × nₚ × (1+nₗ) × (2^(d+1)−1)
Persp tex: u_α = [(1−α)u₀/z₀ + αu₁/z₁] / [(1−α)/z₀ + α/z₁]
```

---

*Document compiled by Claude from Cambridge 3G4 past papers and cribs 2010–2024.*
*Authors of original materials: Andrew Gee & Graham Treece, Cambridge University Engineering Department.*
