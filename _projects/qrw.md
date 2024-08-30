---
layout: distill
title: Quantum Random Walk
description: The behavior of quantum random walks (QRWs) using the Creutz ladder model
img: assets/img/codethoigian.png
importance: 1
category: Physics
disqus_comments: true
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
  - name: Key Contributions
  - name: Visualization
  - name: Conclusion
  - name: Future Directions
  - name: Learn More
---

## Overview
In this project, we explored the behavior of quantum random walks (QRWs) using the Creutz ladder model, a quantum lattice structure known for its localization properties. Our focus was on examining how quantum particles behave within this confined system and comparing it to classical random walks (CRWs).

## Key Contributions
- **Classical and Quantum Walk Regimes**: We validated the expected diffusive behavior in CRWs and the ballistic regime in QRWs. These foundational insights highlight the differences between classical and quantum behavior in random walks.

<div class="row mt-3">
    <div class="col-sm mt-3 mt-md-0">
        {% include figure.liquid loading="eager" path="assets/img/quantumvsclassical.png" class="img-fluid rounded z-depth-1" zoomable=true %}
    </div>
</div>

- **Localization in the Creutz Ladder Model**: By introducing randomness through the Grover coin operator and applying combinatorial methods, we demonstrated that the quantum particle's movement is confined within a specific region. This result enhances our understanding of how structured lattices impact quantum behavior.

<div class="row mt-3">
    <div class="col-sm mt-3 mt-md-0">
        {% include figure.liquid loading="eager" path="assets/img/creutz_ladder_with_coin1.png" class="img-fluid rounded z-depth-1" zoomable=true %}
    </div>
</div>

- **Numerical Simulations**: We conducted numerical simulations that supported our analytical results, showing that the probability of the particle moving beyond a confined range is zero. This confirmation strengthens our findings and demonstrates the effectiveness of our methods.

## Visualization
We visualized the quantum walk on the Creutz ladder, numerically verifying that the particle remains within a confined region. Additionally, we observed recurring patterns in the particle's location probability over time, which opens up new avenues for further research.

<div class="row mt-3">
    <div class="col-sm mt-3 mt-md-0">
        {% include figure.liquid loading="eager" path="assets/img/codethoigian.gif" class="img-fluid rounded z-depth-1" zoomable=true %}
    </div>
</div>

## Conclusion
These findings contribute significantly to our comprehension of QRWs and their potential applications. The demonstrated confinement of quantum walks within the Creutz ladder model suggests promising avenues for future research into controlled quantum systems and their practical implementations.

## Future Directions
- **Exploration of Alternative Lattice Models**: Investigate different lattice structures to understand how they affect QRW behavior and localization.
  
- **Development of Quantum Algorithms**: Utilize the confinement properties observed in this study to design quantum algorithms that require controlled particle movements.
  
- **Experimental Validation**: Conduct real-world experiments to verify the theoretical and numerical results of this study.

## Learn More
For a deeper dive into our work, including detailed equations, proofs, and data, refer to our [full report](https://ducto489.github.io/assets/pdf/Report___Group_1.pdf).

---

*This project was a collaborative effort under the guidance of our mentors and head mentors. The findings represent a preliminary study in the field of quantum random walks, with opportunities for further exploration and learning.*