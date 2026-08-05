---
layout: default
title: Home
nav_order: 0
permalink: /
---

# Introduction to the RSPt DFT code
### Or how I learned to appreciate the small things in life.

## What is RSPt?
- RSPt is a Density Functional Theory (DFT) code, [HK, KS].
- RSPt is an all electron code (no pseudo potentials or projector augmentations).
- RSPt is a Full Potential (FP) code, no assumptions made regarding the shape of the potential.
- RSPt uses Linearized Muffin-Tin Orbitals (LMTO) basis functions [ALin].
All together this make RSPt an FP-LMTO DFT code.

This tutorial will go through the basic steps to set up and run a calculation using RSPt.
We will also cover the very basics of what is required to perform a Dynamical Mean Field Theory (DMFT) calculation.

We will not cover the advanced stuff that you can do in RSPt, such as setting up the perfect basis for your problem,
or how to play around with the core electrons.

Beware: RSPt is not user friendly, nor is it particularly intuitive. You can easily shoot yourself in the foot without noticing it.
Always check that your calculations make sense, RSPt calculations are usually good at converging.
You have to check that the calculation has converged to something reasonable.

# Tutorials
- [Installation](tutorials/installation/install/)
- [First Calculation: FCC Ni](tutorials/FCC_Ni/fcc_Ni/)
- [Nonmagnetic NiO](tutorials/NiO_PM/nio_pm/)
- [Antiferromagnetic NiO](tutorials/NiO_AFM/nio_afm/)
- [LDA+U](tutorials/LDA+U/nio_ldau/)
- [Bandstructure](tutorials/banstructure/bandstructure/)
- [Dynamical Mean Field Theory (DMFT)](tutorials/dmft/dmft/)
- [X-ray Absorption Spectroscopy (XAS)](tutorials/xas/xas/)
- [Tooling](tutorials/tooling/tooling/)
