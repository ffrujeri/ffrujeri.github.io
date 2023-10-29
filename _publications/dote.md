---
title: "DOTE: Rethinking (Predictive) WAN Traffic Engineering"
collection: publications
permalink: /publication/dote
excerpt: "<img src='/images/dote.png' style='display:block; margin:auto;'><br/>We explore a new design point for traffic engineering on wide-area networks (WANs): directly optimizing traffic flow on the WAN using only historical data about traffic demands. Doing so obviates the need to explicitly estimate, or predict, future demands. Our method, which utilizes stochastic optimization, provably converges to the global optimum in well-studied theoretical models. We employ deep learning to scale to large WANs and real-world traffic. Our extensive empirical evaluation on real-world traffic and network topologies establishes that our approach’s TE quality almost matches that of an (infeasible) omniscient oracle, outperforming previously proposed approaches, and also substantially lowers runtimes."
date: "2023-04-01"
venue: 'NSDI (Best Paper Award)'
citation: "Yarin Perry, Felipe Vieira Frujeri, Chaim Hoch, Srikanth Kandula, Ishai Menache, Michael Schapira, Aviv Tamar"
---
We explore a new design point for traffic engineering on wide-area networks (WANs): directly optimizing traffic flow on the WAN using only historical data about traffic demands. Doing so obviates the need to explicitly estimate, or predict, future demands. Our method, which utilizes stochastic optimization, provably converges to the global optimum in well-studied theoretical models. We employ deep learning to scale to large WANs and real-world traffic. Our extensive empirical evaluation on real-world traffic and network topologies establishes that our approach’s TE quality almost matches that of an (infeasible) omniscient oracle, outperforming previously proposed approaches, and also substantially lowers runtimes.

[Paper](https://www.usenix.org/system/files/nsdi23-perry.pdf)
