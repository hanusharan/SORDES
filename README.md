# SORDES

**Adaptive Neural Intelligence for Intent-Driven Robotic Control**

SORDES is a research and engineering project investigating adaptive EEG-based neural control for a virtual robotic exoskeleton. The research focuses on cross-session neural decoding, calibration efficiency, confidence-aware command rejection, and deterministic robotic control.

## Frozen specifications

The `docs/` directory contains the two frozen grounding specifications that define the v1 engineering architecture and pre-registered EEG methodology. These documents are the source of truth for implementation and experimentation.

- `docs/SORDES_EXOSKELETON_SPEC_V1.md` — kinematic exoskeleton, manipulation, command bridge, state machines, and safety specification.
- `docs/SORDES_EEG_METHODOLOGY_V1.md` — neural class vocabulary, BCI Competition IV Dataset 2a methodology, adaptive recalibration experiment, safety criterion, calibration sweep, and latency reporting.

## Implementation principle

Build to the frozen specification rather than improvising requirements. The Blender layer is a kinematic simulation and visualization layer; the EEG/AI layer is the research component. No physical dynamics, clinical efficacy, or hardware validation is implied by the v1 simulation.
