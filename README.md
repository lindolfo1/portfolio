**Guidance and navigation**

CS degree. Kalman filtering, relative orbit guidance, simulation in Trick and Python. Also CNC fabrication and RF hardware.

Houston, TX

<br>

## Guidance and navigation

<br>

### `01` &nbsp; Autonomous Satellite Rendezvous &nbsp; ![year](https://img.shields.io/badge/2026-555?style=flat-square) ![stack](https://img.shields.io/badge/C++_·_Trick-264653?style=flat-square)

> A closed guidance and navigation loop for a chaser satellite closing on a passive target in a 500 km circular orbit, built in Trick. Coupled attitude and relative-state filters, guidance built on Clohessy-Wiltshire transfers, burn execution through a thruster model, and a truth simulation the estimator never gets to see. The chaser starts about a kilometer out at a dispersed position and parks 20 m behind the target.

<div align="center">

https://github.com/user-attachments/assets/9619d791-254c-4c13-80cc-baccf9675db1

*Relative trajectory in the target's LVLH frame, closing through waypoints at 250, 50 and 20 m.*

</div>

- Both vehicles start in a 500 km circular orbit. The chaser is dispersed to a random point at most 45° off the target's rear, gets one copy of the target's state to initialize its filters, and flies on its own sensors from there.
- Attitude determination is a 6-state MEKF over attitude error and gyro bias. It propagates on 10 Hz IMU deltas and is corrected once a second by a paired sun sensor and magnetometer, solved together with TRIAD into a quaternion plus covariance. Some of each correction lands in the bias estimate, so drift between updates shrinks over the run.
- Relative navigation is a 6-state EKF over position and velocity in the target's LVLH frame, propagated at 10 Hz with Clohessy-Wiltshire dynamics plus commanded thrust.
- The two filters are coupled. The rangefinder reports range, azimuth and elevation in the chaser's body frame, so the measurement model has to consume the MEKF's attitude estimate, and attitude error feeds straight into relative nav error.
- Guidance runs every 5 s off the relative estimate. It sweeps transfer times across several orbits, solves the two-burn CW targeting problem for each candidate, and scores delta-v against elapsed time. Candidates whose burn exceeds 5% of the transfer time get thrown out, since the impulsive assumption stops holding there.
- Commanded delta-v goes into a burn queue and the thruster converts it to a duration from vehicle mass and thrust. Guidance re-checks tolerance at the planned arrival time and re-plans against the same waypoint until the chaser is within 1% of waypoint range and slow enough, then steps in to the next one.
- Underneath is the truth simulation: two-body gravity with J2, integrated with RK4 at 5 ms, plus applied thrust. It stays separate from the estimator's model of the world.
- C++ models under Trick with Eigen, one class per file, at flight-like rates (10 Hz filters, 1 Hz sensors, 5 s guidance, 5 ms integration). Python input deck for orbit setup and dispersion, CSV logging, Streamlit viewer for replaying runs.

<br>

### `02` &nbsp; Spacecraft Navigation: Extended Kalman Filter &nbsp; ![year](https://img.shields.io/badge/2026-555?style=flat-square) ![solo](https://img.shields.io/badge/Solo-555?style=flat-square) ![result](https://img.shields.io/badge/600×_over_naive-2a9d8f?style=flat-square)

> Derived and implemented an Extended Kalman Filter for autonomous deep-space navigation: position and velocity from passive optics alone. The sensors are solar angular radius for range and star-tracker azimuth/elevation for bearing. Tested on a highly elliptical asteroid-belt orbit (a = 3.9 AU, e = 0.6).

| EKF vs true orbit | Naive vs EKF |
|---|---|
| ![EKF spacecraft navigation](assets/ekf_plot.png) | ![Naive vs EKF](assets/comparison_plot.png) |
| *EKF estimate vs true orbit with 2σ uncertainty ellipses (exaggerated; at true scale they are sub-pixel). Steady-state error ~2×10⁻⁵ AU.* | *Naive direct inversion peaks at ~0.65 AU near aphelion. The EKF holds 0.000182 AU RMS, roughly 600× better.* |

- State is `[x, y, z, vx, vy, vz]` in heliocentric Cartesian, AU and AU/s throughout.
- Dynamics are two-body gravity, `a = −(GM/r³)r`, propagated with RK4. Covariance propagates through the linearized gravity Jacobian `∂aᵢ/∂rⱼ = GM(3rᵢrⱼ/r⁵ − δᵢⱼ/r³)`, rebuilt each step.
- The measurement model `h(x) = [arcsin(R☉/r), arctan2(y,x), arcsin(z/r)]` maps Cartesian state to sensor space `[ρ, θ, φ]`, with H derived analytically.
- Process noise comes from a piecewise white-noise-acceleration model with position and velocity cross terms, driven by one parameter (σ_a = 1×10⁻¹⁸ AU/s²). Measurement noise is `R = diag[1e-10, 1e-12, 1e-12]` rad².
- Steady-state position error is ~2×10⁻⁵ AU, about 3,000 km, against 0.11 AU RMS for naive per-measurement inversion. Mean NIS of 2.99 against a target of 3.0, so the covariance actually matches the filter's error.
- Observability: uncertainty ellipses elongate along the sun line and grow toward aphelion, matching `dr/dρ = −R☉/sin²ρ`. Range is the weak direction and gets weaker as the solar disk shrinks. Orbital dynamics suppress that, but cannot remove it.
- Three compounding bugs turned up in validation, including a one-step measurement lag that pinned error at a floor no amount of tuning could clear. Seeding the filter with the true state separated model error from bookkeeping error and isolated it.

<br>

## Other work

<br>

### `03` &nbsp; 3D Map of Galactic Dust &nbsp; ![year](https://img.shields.io/badge/2026-555?style=flat-square) ![data](https://img.shields.io/badge/15M_Stars-8338ec?style=flat-square) ![output](https://img.shields.io/badge/3D_·_GIF_·_HTML-555?style=flat-square)

> Reconstructs the 3D dust density field within ~500 pc of the Sun from Gaia DR3 photometry of about 15 million stars. The pipeline queries the archive, pulls per-star reddening, estimates the field, and renders it.

<div align="center">

![Rotating 3D dust map](assets/04-dust-edgeon.gif)

*120-frame rotation of the galactic dust volume. Edge-on view; looking down from above the galactic plane.*

</div>

| | |
|:---:|:---:|
| ![Top-down view](assets/04-dust-topdown.jpg) | ![Oblique view](assets/04-dust-oblique.jpg) |
| *Top-down. Polar projection at galactic plane. Sun at center, 100 pc arcs.* | *Oblique. Looking above the galactic plane toward galactic center.* |

- Chunked ADQL queries across 72 sky regions to get around the Gaia 3M-row response cap, with retry logic, failure logging, and output in columnar Parquet
- Per-star color excess from a hand-fit blue-edge polynomial over the HR diagram, giving the intrinsic-color baseline for reddening
- KD-tree spatial indexing in Cartesian galactic coordinates; a 3D Gaussian kernel reconstructs cumulative reddening, and centered finite differencing turns that into dust density
- Written from scratch, without an existing dust-map library
- Reproduces structural features of the published Bayestar map: foreground dust at ~100 pc and the rise into the Cygnus complex at ~300 pc are both visible

<br>

### `04` &nbsp; Solar Projector Telescopes &nbsp; ![year](https://img.shields.io/badge/2024-555?style=flat-square) ![role](https://img.shields.io/badge/Co--Founder-e76f51?style=flat-square) ![deadline](https://img.shields.io/badge/Apr_8_Eclipse-555?style=flat-square)

> Co-founded a company to design and manufacture solar projector telescopes for the April 8, 2024 total solar eclipse. Concept to multi-unit production in three months.

<div align="center">

![Production unit](assets/02-solar-telescope.jpg)

*Production unit. Focusable front lens, two flat mirrors folding the optical path, eyepiece projecting the solar image onto a semi-transparent rear screen.*

</div>

- Plywood enclosure machined with multi-tool CAM, designed for repeatable batch fabrication
- Folded optical path (focusable front lens, two flat mirrors, eyepiece) projects the solar disk onto a rear semi-transparent screen. The design is safe because of its geometry rather than because of a filter.
- Sunspots resolve clearly on the projected disk, which was the image quality target
- All units shipped before eclipse day

<br>

### `05` &nbsp; 3-Axis CNC Router &nbsp; ![year](https://img.shields.io/badge/2022-555?style=flat-square) ![budget](https://img.shields.io/badge/$250-2a9d8f?style=flat-square) ![solo](https://img.shields.io/badge/Solo-555?style=flat-square)

> Designed and built from scratch in 3 months. 20″ × 10″ × 4″ working envelope; cuts aluminum, acrylic, and wood.

| | |
|:---:|:---:|
| ![Completed router](assets/01-cnc-router.jpeg) | ![Controller enclosure](assets/01-cnc-controller.jpeg) |
| *Completed router with workpiece on the bed. Aluminum frame, linear rails.* | *Custom 3D-printed controller enclosure with CNC shield and stepper drivers.* |

- Aluminum extrusion frame with linear rails on all three axes
- Custom 3D-printed controller enclosure housing the CNC shield, stepper drivers, e-stop, and limit switch terminals
- Fusion 360 CAM, with feeds and speeds tuned per material by test cuts
- Several structural parts cracked before the design settled. The fixes were thicker walls, larger fillets, and ribbed cross sections.

<br>

### `06` &nbsp; 1.42 GHz Radio Telescope &nbsp; ![year](https://img.shields.io/badge/2019-555?style=flat-square) ![result](https://img.shields.io/badge/H--I_Line_Resolved-264653?style=flat-square)

> A backyard hydrogen-line detector built from foil-faced insulation board. Resolved the 21 cm H-I emission line from the Milky Way in a 16-hour transit scan.

<div align="center">

![16-hour drift scan spectrogram](assets/03-drift-scan.png)

*16-hour drift scan. Y-axis: frequency centered on 1420.405 MHz (orange marker). X-axis: time. The horizontal band is H-I emission; drift above and below the marker tracks Doppler shift from galactic rotation.*

</div>

| ![The Horn](assets/03-horn.jpeg) | ![Inside the Horn](assets/03-horn-inside.jpeg) | ![The Receiver](assets/03-receiver.jpeg) |
|:---:|:---:|:---:|
| *Pyramidal horn from foil-faced polyiso panels* | *Waveguide section; probe couples to coax* | *1.42 GHz SAW-filtered LNA + RTL-SDR dongle* |

- Pyramidal horn antenna from foil-faced polyiso panels, with a waveguide section and coax-coupled probe
- RF chain: 1.42 GHz SAW-filtered LNA into an RTL-SDR dongle
- Fixed pointing. Earth's rotation sweeps the beam across the galactic plane, and long integration lets the H-I line come up out of thermal noise.
- Drift above and below the rest frequency (1420.405 MHz) is visible in the spectrogram, tracking Doppler shift from galactic rotation

<br>

## Skills & Tools

| Domain | Details |
|:---|:---|
| **State estimation** | Extended and multiplicative Kalman filters, attitude and gyro bias estimation, covariance propagation, Jacobian linearization, process and measurement noise tuning, NIS consistency checking |
| **Navigation** | Relative navigation in LVLH, Clohessy-Wiltshire dynamics, TRIAD attitude determination, sensor fusion, observability analysis, spherical and Cartesian transforms |
| **Guidance** | Two-burn CW targeting, delta-v and transfer time trades, waypoint sequencing, burn scheduling against a thruster model |
| **Flight dynamics** | Two-body and J2 gravity, RK4 propagation, Keplerian orbits, body and LVLH frame conversions |
| **Simulation** | Trick, multi-rate scheduling, dispersed initial conditions, truth models separated from estimators, run logging and replay |
| **Software** | C++, Eigen, Python, numpy, scipy, matplotlib, Plotly, Parquet, Streamlit |
| **CAD / CAM** | Fusion 360 (design + CAM), 3D printing |
| **Fabrication** | CNC routing, aluminum & wood, plywood machining |
| **Electronics** | Stepper drivers, CNC controllers, RF front ends, SDR |
| **Astronomy** | Gaia archive (ADQL), radio astronomy, photometry |
