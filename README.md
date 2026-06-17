**Mechanical Design, Prototyping & Software**

CS degree · hands-on builder · from CNC fabrication and RF hardware through data pipelines and state estimation

Houston, TX

<br>

## Projects

<br>

### `01` &nbsp; Spacecraft Navigation — Extended Kalman Filter &nbsp; ![year](https://img.shields.io/badge/2026-555?style=flat-square) ![solo](https://img.shields.io/badge/Solo-555?style=flat-square) ![result](https://img.shields.io/badge/120×_over_naive-2a9d8f?style=flat-square)

> Derived and implemented an Extended Kalman Filter for autonomous deep-space position and velocity estimation. Sensor suite: solar angular radius (range proxy) and star-tracker azimuth/elevation (bearing). No GPS. No ground contact. Tested on a highly elliptical asteroid-belt orbit (a = 3.9 AU, e = 0.6) — a geometry that stresses any range-based estimator at aphelion.

| | |
|:---:|:---:|
| ![EKF spacecraft navigation](ekf_plot.png) | ![Naive vs EKF](comparison_plot.png) |
| *EKF estimate vs true orbit with 2σ uncertainty ellipses. Max error ~0.05 AU at aphelion.* | *Naive direct inversion peaks at ~6.2 AU error at aphelion. EKF stays below 0.05 AU — 120× improvement.* |

- **State vector** `[x, y, z, vx, vy, vz]` in heliocentric Cartesian; units AU and AU/s throughout
- **Dynamics model** — Newton's law of gravitation `a = −(GM/r³)r̂` propagates state; F is the linearised gravity Jacobian `∂aᵢ/∂rⱼ = GM(3rᵢrⱼ/r⁵ − δᵢⱼ/r³)` recomputed each step for covariance propagation — the standard EKF separation of nonlinear state propagation from linearised uncertainty propagation
- **Measurement function** — nonlinear `h(x) = [arcsin(R☉/r), arctan2(y,x), arcsin(z/r)]` maps Cartesian state to sensor space `[ρ, θ, φ]`; Jacobian H derived analytically and recomputed each step
- **Observability analysis** — residual ~0.05 AU error at aphelion is a fundamental limit: `dr/dρ = −R☉/sin²(ρ)` diverges as the sun subtends a smaller angle; uncertainty ellipses in the covariance plot correctly inflate in the radial direction, confirming the filter knows where it is uncertain
- **Tuning** — process noise Q `diag[1e-6, 1e-6, 1e-6, 1e-8, 1e-8, 1e-8]`; measurement noise R `diag[1e-10, 1e-12, 1e-12]` (sensor variances in rad²); initial P on diagonal

<br>

### `02` &nbsp; 3D Map of Galactic Dust &nbsp; ![year](https://img.shields.io/badge/2026-555?style=flat-square) ![data](https://img.shields.io/badge/15M_Stars-8338ec?style=flat-square) ![output](https://img.shields.io/badge/3D_·_GIF_·_HTML-555?style=flat-square)

> From-scratch reconstruction of the 3D dust density field within ~500 pc of the Sun from Gaia DR3 photometry of ~15 million stars. End-to-end pipeline: archive query → reddening extraction → spatial estimation → interactive visualization.

<div align="center">

![Rotating 3D dust map](assets/04-dust-edgeon.gif)

*120-frame rotation of the galactic dust volume. Edge-on view; looking down from above the galactic plane.*

</div>

| | |
|:---:|:---:|
| ![Top-down view](assets/04-dust-topdown.jpg) | ![Oblique view](assets/04-dust-oblique.jpg) |
| *Top-down. Polar projection at galactic plane. Sun at center, 100 pc arcs.* | *Oblique. Looking above the galactic plane toward galactic center.* |

- Chunked ADQL queries across 72 sky regions to work around the Gaia 3M-row response cap; retry logic and failure logging; output in columnar Parquet
- Per-star color excess from a hand-fit blue-edge polynomial over the HR diagram (intrinsic-color baseline for reddening)
- KD-tree spatial indexing in Cartesian galactic coordinates; 3D Gaussian kernel weighting reconstructs cumulative reddening; centered finite differencing yields dust density
- No off-the-shelf dust-map library used
- Reproduces structural features of the published Bayestar map: foreground dust at ~100 pc and rise into the Cygnus complex at ~300 pc both visible

<br>

### `03` &nbsp; Solar Projector Telescopes &nbsp; ![year](https://img.shields.io/badge/2024-555?style=flat-square) ![role](https://img.shields.io/badge/Co--Founder-e76f51?style=flat-square) ![deadline](https://img.shields.io/badge/Apr_8_Eclipse-555?style=flat-square)

> Co-founded a venture to design and manufacture solar projector telescopes for the April 8, 2024 total solar eclipse. Concept to multi-unit production in three months.

<div align="center">

![Production unit](assets/02-solar-telescope.jpg)

*Production unit. Focusable front lens, two flat mirrors folding the optical path, eyepiece projecting the solar image onto a semi-transparent rear screen.*

</div>

- Plywood enclosure machined with multi-tool CAM; designed from day one for repeatable batch fabrication
- Folded optical path (focusable front lens → two flat mirrors → eyepiece) projects the solar disk onto a rear semi-transparent screen — inherently safe by form factor, not by filter
- Image quality target met: sunspots clearly resolved on the projected disk
- All units shipped before eclipse day

<br>

### `04` &nbsp; 3-Axis CNC Router &nbsp; ![year](https://img.shields.io/badge/2022-555?style=flat-square) ![budget](https://img.shields.io/badge/$250-2a9d8f?style=flat-square) ![solo](https://img.shields.io/badge/Solo-555?style=flat-square)

> Designed and built from scratch in 3 months. 20″ × 10″ × 4″ working envelope; cuts aluminum, acrylic, and wood.

| | |
|:---:|:---:|
| ![Completed router](assets/01-cnc-router.jpeg) | ![Controller enclosure](assets/01-cnc-controller.jpeg) |
| *Completed router with workpiece on the bed. Aluminum frame, linear rails.* | *Custom 3D-printed controller enclosure with CNC shield and stepper drivers.* |

- Aluminum extrusion frame with linear rails on all three axes
- Custom 3D-printed controller enclosure housing the CNC shield, stepper drivers, e-stop, and limit switch terminals
- Fusion 360 CAM with feeds and speeds tuned per material by test cuts
- Iterated structural parts through several failures; cracked brackets became an education in wall thickness, fillet sizing, and ribbed cross sections

<br>

### `05` &nbsp; 1.42 GHz Radio Telescope &nbsp; ![year](https://img.shields.io/badge/2019-555?style=flat-square) ![result](https://img.shields.io/badge/H--I_Line_Resolved-264653?style=flat-square)

> A backyard hydrogen-line detector built from foil-faced insulation board. Resolved the 21 cm H-I emission line from the Milky Way in a 16-hour transit scan.

<div align="center">

![16-hour drift scan spectrogram](assets/03-drift-scan.png)

*16-hour drift scan. Y-axis: frequency centered on 1420.405 MHz (orange marker). X-axis: time. The horizontal band is H-I emission; drift above/below the marker tracks Doppler shift from galactic rotation.*

</div>

| ![The Horn](assets/03-horn.jpeg) | ![Inside the Horn](assets/03-horn-inside.jpeg) | ![The Receiver](assets/03-receiver.jpeg) |
|:---:|:---:|:---:|
| *Pyramidal horn from foil-faced polyiso panels* | *Waveguide section; probe couples to coax* | *1.42 GHz SAW-filtered LNA + RTL-SDR dongle* |

- Pyramidal horn antenna from foil-faced polyiso panels; waveguide section with coax-coupled probe
- RF chain: 1.42 GHz SAW-filtered LNA → RTL-SDR dongle
- Fixed pointing; Earth's rotation swept the beam across the galactic plane; long integration lets the H-I line emerge from thermal noise
- Drift above and below the rest frequency (1420.405 MHz) visible in the spectrogram, tracking Doppler shift from galactic rotation

<br>

## Skills & Tools

| Domain | Details |
|:---|:---|
| **State estimation** | Extended Kalman Filter, covariance propagation, Jacobian linearisation, observability analysis |
| **Navigation** | Heliocentric orbital mechanics, sensor fusion, spherical ↔ Cartesian coordinate transforms |
| **Dynamics modelling** | Newtonian gravity, Keplerian orbit integration, process and measurement noise tuning |
| **Software** | Python, numpy, scipy, matplotlib, Plotly, Parquet |
| **CAD / CAM** | Fusion 360 (design + CAM), 3D printing |
| **Fabrication** | CNC routing, aluminum & wood, plywood machining |
| **Electronics** | Stepper drivers, CNC controllers, RF front ends, SDR |
| **Astronomy** | Gaia archive (ADQL), radio astronomy, photometry |
