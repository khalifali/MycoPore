# MycoPore

**Computational modelling and design of fungal porous materials**

MycoPore is a research code and workflow for studying how the morphology of fungal and mycelium-based porous materials controls their effective transport and functional properties.

The long-term goal is to connect

**Growth and morphology** → **pore geometry** → **pore-scale physics** → **effective properties** → **material performance**

with an initial focus on **acoustics**, followed by heat and mass transport, permeability, and eventually inverse design.

---

## Scientific idea

Fungal and mycelium-based materials possess a hierarchical pore structure created by interacting hyphae, substrate particles, branches, constrictions, voids, and potentially strong anisotropy.

Rather than treating the material only through bulk empirical correlations, MycoPore aims to resolve and quantify the connection between microstructure and macroscopic material response.

For acoustics, the intended modelling chain is

**Synthetic fungal geometry** → **pore-scale CFD** → **effective parameters** → **JCAL model** → **sound absorption**

where

- $\phi$ is open porosity,
- $\sigma$ is airflow resistivity,
- $\alpha_\infty$ is high-frequency tortuosity,
- $\Lambda$ is the viscous characteristic length,
- $\Lambda'$ is the thermal characteristic length,
- $\alpha(f)$ is the frequency-dependent sound absorption coefficient.

A later stage will directly solve thermoviscous acoustics in resolved pore geometries for selected validation cases.

---

## Main research questions

MycoPore will initially address questions such as:

1. How do fungal morphology descriptors control permeability and airflow resistivity?
2. How do branching, connectivity, tortuosity, orientation, and constrictions affect acoustic dissipation?
3. Which geometric descriptors are sufficient to predict effective poroacoustic parameters?
4. How strongly does anisotropic fungal growth influence directional transport and acoustics?
5. Can synthetic fungal networks reproduce the acoustic behaviour of real mycelium-based materials?
6. Can morphology be inversely designed for a target acoustic or transport response?

---

## Modelling strategy

### Geometry generation

The first geometries will be synthetic and progressively increase in complexity:

1. single cylindrical pore,
2. parallel pores,
3. tortuous pore,
4. branching pore network,
5. random fibre / hyphal network,
6. biologically inspired fungal network,
7. fungal network embedded in or grown through a particulate substrate.

A fungal network can initially be represented as connected cylindrical hyphal segments defined by

$$
(\mathbf{x}_i,\mathbf{x}_j,d_h)
$$

for each segment, with branching rules controlling orientation, length, diameter, and connectivity.

The geometry generator should eventually support periodic representative volume elements (RVEs).

---

## Pore-scale CFD

### Computational domain

The solid fungal structure is treated as impermeable; the connected gas phase defines the computational domain:

$$
\Omega_f = \Omega_{\mathrm{RVE}} \setminus \Omega_{\mathrm{solid}}.
$$

The initial meshing workflow is

**Python geometry** → **STL/OBJ** → **blockMesh** → **snappyHexMesh** → **checkMesh**

The background mesh is Cartesian, with local refinement near hyphae, junctions, and narrow gaps.

For early Stokes-flow calculations, the priority is geometric convergence rather than boundary-layer prism meshes.

### First solver

The first CFD problem is steady creeping flow:

$$
-\nabla p + \mu\nabla^2\mathbf{u}=0,
$$

$$
\nabla\cdot\mathbf{u}=0.
$$

Boundary conditions:

- pressure difference between inlet and outlet,
- no-slip velocity on fungal surfaces,
- periodic boundaries in transverse directions where possible.

From the volumetric flow rate

$$
Q=\int_A u_n\,\mathrm{d}A,
$$

the permeability is obtained from Darcy's law:

$$
K=\frac{\mu LQ}{A\Delta p},
$$

and the airflow resistivity is

$$
\sigma=\frac{\mu}{K}.
$$

The same RVE can be driven independently in $x$, $y$, and $z$ to quantify anisotropy.

---

## Literature context: pore-scale CFD with idealised particle and fibre geometries

A substantial pore-scale CFD literature supports the use of idealised geometric primitives to represent porous microstructures. This is important for MycoPore because the fungal skeleton does not need to be reconstructed from CT data in the first stage; it can be generated computationally from connected particles, elongated particles, or fibres and the flow can be resolved through the remaining pore space.

### Sphere-based porous media

**Martys, Torquato & Bentz (1994)** studied transport in model porous media generated from random overlapping and non-overlapping spheres. The work is an early example showing that collections of simple primitives can be used as synthetic porous microstructures and still yield meaningful effective transport properties.

**Pan, Hilpert & Miller (2001)** simulated flow through random sphere packings using lattice-Boltzmann and pore-network methods. Their work established direct links between explicitly resolved particle geometry and permeability.

**Zaman & Jalali (2010)** used CFD to resolve flow through random monodisperse sphere packings containing up to thousands of particles and extracted Darcy permeability directly from the simulated interparticle flow.

These studies establish that an explicit sphere assembly is a standard and physically useful pore-scale representation.

### Overlapping spheres and multi-sphere approximations

Some studies deliberately use overlapping spheres or multi-sphere clusters to represent a continuous or irregular solid rather than literal separate particles.

**Lane et al. (2013)** used overlapping spheres as a surrogate microstructure for consolidated porous materials and compared numerical flow approaches.

**Kerimov et al. (2018)** discussed multi-sphere representations as a practical route for approximating irregular particle geometries.

This is directly relevant to the first MycoPore fallback representation, in which a hyphal branch can be approximated by a chain of overlapping resolved spheres. The important numerical requirement is that the immersed-boundary forcing represents the **union of the solid geometry**, so overlap is not double counted.

### Spheroids, ellipsoids and superquadric particles

Non-spherical particle geometry has also been used extensively for pore-scale permeability studies.

**Kerimov et al. (2018)** generated porous structures containing spheres, prolate spheroids, oblate spheroids and irregular convex particles and solved the pore flow using lattice Boltzmann. A central result was that particle shape can affect permeability strongly even when porosity changes only modestly.

**Xu et al. (2022)** studied dense mono- and polydisperse spheroidal porous media and quantified the effects of aspect ratio, orientation and size distribution on tortuosity and permeability.

A **2025 *Transport in Porous Media*** study generated ellipsoidal beds using superquadric particle representations in LIGGGHTS, converted the structures to pore geometries, and then performed pore-scale CFD in OpenFOAM. This is one of the closest technical precedents for MycoPore:

```text
DEM superquadrics → explicit porous geometry → OpenFOAM pore-scale CFD → permeability
```

A **2026 *Computers & Geotechnics*** study likewise investigated permeability of superquadric granular materials using coupled DEM–LBM.

These works support the planned use of elongated superellipsoids as a compact representation of approximately cylindrical hyphal segments.

### Fibrous porous media

Fibrous porous media are probably the closest physical analogue to fungal hyphae.

**Koponen et al. (1998)** calculated creeping flow through three-dimensional random fibre webs and predicted permeability from the synthetic fibre geometry.

**Nabovati et al. (2009)** generated synthetic three-dimensional random fibre media and used lattice Boltzmann to study the effects of fibre diameter, aspect ratio, curvature and orientation on permeability.

**Yazdchi, Srivastava & Luding (2011)** solved Stokes flow through periodic fibrous structures with circular, elliptical and square cross-sections and quantified the dependence of permeability on fibre geometry and orientation.

These studies are especially important for MycoPore because they show that realistic porous transport can be investigated using **synthetically generated fibre networks without requiring CT reconstruction**.

### Implications for MycoPore

The literature supports three levels of idealisation:

1. **sphere assemblies** for baseline and validation studies,
2. **overlapping sphere chains / multi-sphere bodies** for approximate fibre or hyphal geometry,
3. **elongated spheroids or superellipsoids** for a more compact and smoother representation of individual hyphal segments.

MycoPore will therefore investigate the workflow

```text
biologically inspired network generator
        ↓
connected oriented superellipsoidal / particle segments
        ↓
resolved immersed-boundary pore-scale CFD
        ↓
K, sigma, tortuosity and local flow fields
        ↓
poroacoustic homogenisation
```

The intended distinction from much of the granular literature is that the particles are not necessarily physical grains. They are geometric primitives representing a **connected fungal skeleton**. The network connectivity is retained separately as a graph, allowing the same morphology to be used later for growth models, structural mechanics, breakage and graph-based machine learning.

For validation, selected geometries will also be solved with a conventional body-fitted mesh. Agreement between the immersed-boundary and body-fitted solutions will determine whether the particle representation is sufficiently accurate for large morphology campaigns.

### Core references to follow

- Martys, N. S., Torquato, S. & Bentz, D. P. (1994). *Universal scaling of fluid permeability for sphere packings*. Physical Review E.
- Koponen, A. et al. (1998). *Permeability of Three-Dimensional Random Fiber Webs*. Physical Review Letters.
- Pan, C., Hilpert, M. & Miller, C. T. (2001). Pore-scale modelling of flow through sphere packings.
- Nabovati, A., Llewellin, E. W. & Sousa, A. C. M. (2009). Permeability of three-dimensional fibrous porous media by lattice Boltzmann.
- Yazdchi, K., Srivastava, S. & Luding, S. (2011). Microstructural effects on permeability of fibrous porous media.
- Kerimov, A. et al. (2018). Pore-scale simulations examining particle-shape effects on permeability.
- Xu et al. (2022). Tortuosity and permeability of dense spheroidal porous media.
- *Microstructure Simulation to Predict the Influence of Particle Properties on Permeability of Granular Porous Media* (Transport in Porous Media, 2025).
- *Investigation of permeability of superquadric granular materials based on discrete element–lattice Boltzmann method* (Computers & Geotechnics, 2026).

## Validation ladder

No biologically complex geometry should be used before the numerical workflow passes simple benchmark cases.

### Case 01 — Single tube

Validate against Hagen-Poiseuille flow:

$$
Q=\frac{\pi R^4\Delta p}{8\mu L}.
$$

Target: mesh-independent agreement with the analytical solution.

### Case 02 — Parallel tubes

Verify additive hydraulic conductance.

### Case 03 — Tortuous tube

Quantify the effect of path length and tortuosity.

### Case 04 — Branching network

Investigate junctions, constrictions, and flow redistribution.

### Case 05 — Random fibre network

Move toward a porous fibrous morphology.

### Case 06 — Synthetic fungal network

Apply the complete meshing and homogenisation workflow to a generated hyphal network.

---

## Acoustic modelling roadmap

### Phase A — Homogenised poroacoustics

The first acoustic model will use pore-scale calculations to obtain effective parameters rather than resolving acoustic waves in every pore.

Instead, pore-scale calculations will be used to obtain effective parameters such as

$$
\phi,\quad\sigma,\quad\alpha_\infty,\quad\Lambda,\quad\Lambda'.
$$

These will be supplied to a Johnson-Champoux-Allard / JCAL-type model to calculate frequency-dependent effective density, bulk modulus, impedance, and sound absorption.

This makes it possible to predict the behaviour of centimetre-scale panels from micrometre-scale geometry at practical computational cost.

### Phase B — Direct thermoviscous acoustics

Selected pore geometries will later be solved directly using the linearised compressible Navier-Stokes and energy equations in the frequency domain.

The intended unknowns are the complex perturbations

$$
\hat p,\qquad\hat{\mathbf u},\qquad\hat T.
$$

A future OpenFOAM solver may split each field into real and imaginary parts:

$$
p=p_R+ip_I,
$$

with analogous representations for velocity and temperature.

Direct pore-scale thermoviscous simulations will primarily be used for:

- validation of homogenised models,
- studying local dissipation mechanisms,
- identifying the role of constrictions and branching,
- testing regimes where JCAL assumptions may become insufficient.

---

## Planned repository structure

```text
MycoPore/
├── geometry/
│   ├── generators/
│   └── examples/
├── poreFlow/
│   ├── 01_singleTube/
│   ├── 02_parallelTubes/
│   ├── 03_tortuousTube/
│   ├── 04_branchingNetwork/
│   ├── 05_randomFibres/
│   └── 06_fungalNetwork/
├── homogenization/
│   ├── permeability/
│   ├── tortuosity/
│   └── characteristicLengths/
├── acoustics/
│   ├── jcal/
│   └── thermoviscous/
├── transport/
├── growth/
├── ml/
├── validation/
├── docs/
└── README.md
```

The structure is intentionally broader than acoustics so that the same geometries can later be used for heat transfer, gas diffusion, moisture transport, and other functional properties.

---

## First milestone

The first milestone is deliberately small:

> **Generate, mesh, and solve pressure-driven Stokes flow through a validated synthetic pore geometry and automatically extract permeability and airflow resistivity.**

The first implementation sequence is:

1. implement the single-tube geometry,
2. create a reproducible OpenFOAM 13 mesh,
3. run steady creeping flow,
4. compare $Q$ and $K$ against Hagen-Poiseuille,
5. perform a mesh-convergence study,
6. automate post-processing,
7. extend the same workflow to a branching geometry.

Only after this workflow is robust should the project move to generated fungal networks.

---

## Longer-term vision

The broader MycoPore workflow is intended to evolve toward

**Growth conditions** → **3-D fungal morphology** → **multiphysics properties** → **physics/ML surrogate** → **inverse material design**

Possible future outputs include:

- permeability and airflow resistivity,
- poroacoustic absorption,
- effective thermal conductivity,
- gas and moisture diffusivity,
- anisotropic transport tensors,
- morphology-property surrogate models,
- inverse-designed fungal structures.

The central scientific objective is not merely to simulate fungal materials, but to determine **which aspects of fungal morphology control macroscopic function and how those aspects can be engineered**.

---

## Status

Project initiated: September 2026.

This README describes the planned research workflow. The repository currently contains the project roadmap; the listed solvers, cases, and directory tree are planned work.

Current stage: **Phase 0 — geometry, meshing, and pore-flow validation.**
