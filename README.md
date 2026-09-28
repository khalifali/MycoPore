# MycoPore

**Computational modelling and design of fungal porous materials**

MycoPore is a research code and workflow for studying how the morphology of fungal and mycelium-based porous materials controls their effective transport and functional properties.

The long-term goal is to connect

[
	ext{growth / morphology}
ightarrow
	ext{pore geometry}
ightarrow
	ext{pore-scale physics}
ightarrow
	ext{effective properties}
ightarrow
	ext{material performance}
]

with an initial focus on **acoustics**, followed by heat and mass transport, permeability, and eventually inverse design.

---

## 1. Scientific idea

Fungal and mycelium-based materials possess a hierarchical pore structure created by interacting hyphae, substrate particles, branches, constrictions, voids, and potentially strong anisotropy.

Rather than treating the material only through bulk empirical correlations, MycoPore aims to resolve and quantify the connection between microstructure and macroscopic material response.

For acoustics, the intended modelling chain is

[
oxed{
	ext{synthetic fungal geometry}
ightarrow
	ext{pore-scale CFD}
ightarrow
(phi,sigma,alpha_infty,Lambda,Lambda')
ightarrow
	ext{JCAL}
ightarrow
alpha(f)
}
]

where

- (phi) is open porosity,
- (sigma) is airflow resistivity,
- (alpha_infty) is high-frequency tortuosity,
- (Lambda) is the viscous characteristic length,
- (Lambda') is the thermal characteristic length,
- (alpha(f)) is the frequency-dependent sound absorption coefficient.

A later stage will directly solve thermoviscous acoustics in resolved pore geometries for selected validation cases.

---

## 2. Main research questions

MycoPore will initially address questions such as:

1. How do fungal morphology descriptors control permeability and airflow resistivity?
2. How do branching, connectivity, tortuosity, orientation, and constrictions affect acoustic dissipation?
3. Which geometric descriptors are sufficient to predict effective poroacoustic parameters?
4. How strongly does anisotropic fungal growth influence directional transport and acoustics?
5. Can synthetic fungal networks reproduce the acoustic behaviour of real mycelium-based materials?
6. Can morphology be inversely designed for a target acoustic or transport response?

---

## 3. Modelling strategy

### 3.1 Geometry generation

The first geometries will be synthetic and progressively increase in complexity:

1. single cylindrical pore,
2. parallel pores,
3. tortuous pore,
4. branching pore network,
5. random fibre / hyphal network,
6. biologically inspired fungal network,
7. fungal network embedded in or grown through a particulate substrate.

A fungal network can initially be represented as connected cylindrical hyphal segments defined by

[
(mathbf{x}_i,mathbf{x}_j,d_h)
]

for each segment, with branching rules controlling orientation, length, diameter, and connectivity.

The geometry generator should eventually support periodic representative volume elements (RVEs).

---

## 4. Pore-scale CFD

### 4.1 Computational domain

The solid fungal structure is treated as an impermeable solid and only the connected gas phase is meshed:

[
Omega_f = Omega_{mathrm{RVE}} - Omega_{mathrm{solid}}.
]

The initial meshing workflow is

[
	ext{Python geometry}
ightarrow
	ext{STL/OBJ}
ightarrow
	ext{blockMesh}
ightarrow
	ext{snappyHexMesh}
ightarrow
	ext{checkMesh}.
]

The background mesh is Cartesian, with local refinement near hyphae, junctions, and narrow gaps.

For early Stokes-flow calculations, the priority is geometric convergence rather than boundary-layer prism meshes.

### 4.2 First solver

The first CFD problem is steady creeping flow:

[
-
abla p + mu 
abla^2 mathbf{u}=0,
]

[

ablacdotmathbf{u}=0.
]

Boundary conditions:

- pressure difference between inlet and outlet,
- no-slip velocity on fungal surfaces,
- periodic boundaries in transverse directions where possible.

From the volumetric flow rate

[
Q=int_A u_n,dA,
]

the permeability is obtained from Darcy's law:

[
K=rac{mu LQ}{ADelta p},
]

and the airflow resistivity is

[
oxed{sigma=rac{mu}{K}}.
]

The same RVE can be driven independently in (x), (y), and (z) to quantify anisotropy.

---

## 5. Validation ladder

No biologically complex geometry should be used before the numerical workflow passes simple benchmark cases.

### Case 01 — Single tube

Validate against Hagen-Poiseuille flow:

[
Q=rac{pi R^4}{8mu L}Delta p.
]

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

## 6. Acoustic modelling roadmap

### Phase A — Homogenised poroacoustics

The first acoustic model will **not** resolve acoustic waves directly inside every pore.

Instead, pore-scale calculations will be used to obtain effective parameters such as

[
phi,quad
sigma,quad
alpha_infty,quad
Lambda,quad
Lambda'.
]

These will be supplied to a Johnson-Champoux-Allard / JCAL-type model to calculate frequency-dependent effective density, bulk modulus, impedance, and sound absorption.

This makes it possible to predict the behaviour of centimetre-scale panels from micrometre-scale geometry at practical computational cost.

### Phase B — Direct thermoviscous acoustics

Selected pore geometries will later be solved directly using the linearised compressible Navier-Stokes and energy equations in the frequency domain.

The intended unknowns are the complex perturbations

[
hat p,qquad hat{mathbf u},qquad hat T.
]

A future OpenFOAM solver may split each field into real and imaginary parts:

[
p=p_R+ip_I,
]

with analogous representations for velocity and temperature.

Direct pore-scale thermoviscous simulations will primarily be used for:

- validation of homogenised models,
- studying local dissipation mechanisms,
- identifying the role of constrictions and branching,
- testing regimes where JCAL assumptions may become insufficient.

---

## 7. Planned repository structure

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

## 8. First milestone

The first milestone is deliberately small:

> **Generate, mesh, and solve pressure-driven Stokes flow through a validated synthetic pore geometry and automatically extract permeability and airflow resistivity.**

The first implementation sequence is:

1. implement the single-tube geometry,
2. create a reproducible OpenFOAM 13 mesh,
3. run steady creeping flow,
4. compare (Q) and (K) against Hagen-Poiseuille,
5. perform a mesh-convergence study,
6. automate post-processing,
7. extend the same workflow to a branching geometry.

Only after this workflow is robust should the project move to generated fungal networks.

---

## 9. Longer-term vision

The broader MycoPore workflow is intended to evolve toward

[
oxed{	ext{growth conditions}}
ightarrow
oxed{	ext{3-D fungal morphology}}
ightarrow
oxed{	ext{multiphysics properties}}
ightarrow
oxed{	ext{physics/ML surrogate}}
ightarrow
oxed{	ext{inverse material design}}.
]

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

Current stage: **Phase 0 — geometry, meshing, and pore-flow validation.**
