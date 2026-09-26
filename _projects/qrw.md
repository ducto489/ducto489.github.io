---
layout: distill
title: Quantum Random Walk
description: Studying confinement and localization in quantum random walks on the Creutz ladder
img: assets/img/codethoigian-png.png
importance: 1
category: Physics
disqus_comments: false
date: 2024-08-29
featured: true

authors:
  - name: Nguyễn Minh Đức
    url: "https://ducto489.github.io/"
    affiliations:
      name: VNU-HCMUS
  - name: Hồ Trần Khánh Linh
    affiliations:
      name: HNEU
  - name: Nguyễn Duy Tân

toc:
  - name: Overview
  - name: Key Results
  - name: Visualization
  - name: Conclusion
  - name: Future Directions
  - name: Learn More
---

## Overview

In this project, we studied classical and quantum random walks on the Creutz ladder, a lattice model with localization properties. Our goal was to compare the spreading behavior of classical random walks (CRWs) and quantum random walks (QRWs), then investigate how the ladder structure can confine the quantum walker.

The implementation and supporting material are available in the [QRW-MaSSP2024 repository](https://github.com/ducto489/QRW-MaSSP2024).

## Key Results

- **Classical versus quantum spreading:** the simulations reproduce the expected diffusive behavior for CRWs and ballistic spreading for QRWs.
- **Confinement on the Creutz ladder:** using a Grover coin and combinatorial arguments, we found conditions under which the walk remains confined to a bounded region.
- **Numerical verification:** simulations support the analytical argument by showing zero probability outside the predicted confined range for the cases studied.

<div class="row mt-3">
    <div class="col-sm mt-3 mt-md-0">
        {% include figure.liquid loading="eager" path="assets/img/quantumvsclassical.png" class="img-fluid rounded z-depth-1" zoomable=true %}
    </div>
</div>

<div class="row mt-3">
    <div class="col-sm mt-3 mt-md-0">
        {% include figure.liquid loading="eager" path="assets/img/creutz_ladder_with_coin1.png" class="img-fluid rounded z-depth-1" zoomable=true %}
    </div>
</div>

## Visualization

The animation below visualizes the probability distribution of the quantum walker over time. In the simulated regime, the walker remains within the expected confined region and exhibits recurring probability patterns.

<div class="row mt-3">
    <div class="col-sm mt-3 mt-md-0">
        {% include figure.liquid loading="eager" path="assets/img/codethoigian.gif" class="img-fluid rounded z-depth-1" zoomable=true %}
    </div>
</div>

## Conclusion

The project provides a compact analytical and numerical study of confinement in a Creutz-ladder quantum walk. The results are best viewed as a preliminary investigation rather than a general statement about all quantum-walk systems.

## Future Directions

- Study other lattice geometries and coin operators.
- Characterize which parameters preserve or break confinement.
- Compare the analytical predictions with larger numerical experiments.
- Explore whether the confinement mechanism can be useful in controlled quantum-information protocols.

## Learn More

For the derivations, proofs, and numerical details, see the [full report](/assets/pdf/Report___Group_1.pdf).
