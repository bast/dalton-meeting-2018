name: inverse
layout: true
class: middle, inverse

---

# What is cooking in the XC kitchen?

## [Radovan Bast](http://bast.fr)

### [NeIC](https://neic.nordforsk.org)/ [UiT The Arctic University of Norway](https://uit.no)

Text is free to share and remix under [CC-BY-SA-4.0](https://creativecommons.org/licenses/by-sa/4.0/).

---

layout: false

## Goals for this brief overview .blue[XC and density libraries]

- Status
- Vision
- Roadmap
- Challenges
- Lessons learned

---

## What XC codes do

They integrate:

- the XC energy (derivatives)
- the XC potential matrix (derivatives)

### Recipe in "Python notation" (this is not how it is implemented)

```python
grid = construct_grid(basis, atom_centers, atom_charges)
e_xc, v_xc = xc_code(basis, density_matrices, perturbations, grid)
```

---

## In more detail in "Python notation"

(This is not how it is implemented)

```python
grid = construct_grid(basis, atom_centers, atom_charges)

e_xc = 0.0
v_xc = [[0.0 for k in chi] for l in chi]
for (r, w) in grid:

    # compute basis functions in position r
    chi = compute_basis_functions(r)

    # compute density
    density = 0.0
    for chi_k in chi:
        for chi_l in chi:
            density += chi_k*density_matrix[k][l]*chi_l

    # accumulate XC energy
    e_xc += w*xc_derv(density, 0)

    # accumulate XC potential matrix elements
    for chi_k in chi:
        for chi_l in chi:
            v_xc[k][l] += w*chi_k*xc_derv(density, 1)*chi_l
```

- In reality apply tricks and more steps to avoid cubic scaling.

---

## History (may be imprecise)

- CADPAC DFT code
- Dalton DFT implementation (Trygve)
- DIRAC DFT implementation based on Dalton (in collaboration with Dalton/Trygve)
- Dalton DFT rewrite (Pawel)
- LSDalton port based on Dalton
- DIRAC fast-DFT rewrite (never made it to master)
- Rewrite in DIRAC DFT (similar algo to LSDalton; but coded independently)
- XCint
- Multiple rewrites: Fortran 90, later C++
- Realization that it needs to be split up
- WIP: splitting things up

---

## Vision

Plug-n-play libraries for:
- densities and density derivatives
- numerical integration grids
- XC contributions
- visualization post-processing

### .blue[Libraries that do one thing, one thing only, and hopefully well]

## Motivation

- separate concerns
- isolate treatment of point group symmetry to the computation of basis functions and densities
- make the integrator independent of point group symmetry and Hamiltonian
- make the integrator independent of Gaussian/Slater type basis functions
- speed up visualization by using efficient density evaluation libraries

---

### XCFun

- functional derivatives (https://github.com/dftlibs/xcfun)

### XCint

- assemble and integrate (https://github.com/dftlibs/xcint)

### Numgrid

- generate molecular grids (https://github.com/dftlibs/numgrid)

### Balboa

- compute basis function derivatives on a grid (https://github.com/bast/balboa)

### Rhodeo

- compute densities and derivatives (coming soon)

---

## Capabilities and status 1/2

### XCFun

- in principle arbitrary-order functional derivatives
- widely used
- maintenance is a challenge: we need to learn the code
- documentation is outdated

### XCint

- up to 5th order electric derivatives
- up to 4th order geometric derivatives
- good scaling and screening
- currently only used in Dalton/OpenRSP and pilot tests in Stockholm-based codes
- interfaces need documentation

---

## Capabilities status 2/2

### Numgrid

- generate molecular grids (https://github.com/dftlibs/numgrid)
- used in XCint, Dalton current densities, DIRAC (not master), frozen density embedding,
  Stockholm-based codes, and students
- interfaces well documented

### Balboa

- arbitrary-order geometric derivatives of (Gaussian) basis functions
- not used outside of "my" library ecosystem
- needs more exposure

---

## Licenses

- XCFun: LGPL-v3.0, soon MPL-v2.0
- XCint: MPL-v2.0
- Numgrid: MPL-v2.0
- Balboa: MPL-v2.0
- Rhodeo: to be written but will be MPL-v2.0

### MPL-v2.0

- share-alike (like GPL and LGPL)
- compatible with close sourced code (unlike GPL)
- allows static linking (unlike LGPL)

---

## Lessons learned

- Create Python interfaces
- For Python interfaces use [CFFI](https://cffi.readthedocs.io)
  or [pybind11](https://pybind11.readthedocs.io) (depending on whether main interface is C or C++)
- Fortran codes: create Python interfaces with [CFFI](https://cffi.readthedocs.io)
- Test interfaces with Python
- Upload libraries to PyPI
- Document the API really well so that you can delegate work
- Make your code citeable (Zenodo)
- Design the API so that you can avoid parallelization: let the caller do the parallelization

---

## Roadmap

### Short-term (OpenRSP release)

- Conclude fragmentation of XCint into smaller libraries
- Document XCint API
- Document XCFun API
- XCint will become a thin shell around the other libraries

### Advanced user support (ongoing)

- Speed up GIMIC code which will use Balboa and Rhodeo libraries
- New input parser
- Web interface for driving calculations and post-processing

---

## Long-term challenges in XC integration

### Current solution

- dense matrices
- parallelization is over the grid points

### Outlook

- sparse matrices
- parallelization over basis set blocks
