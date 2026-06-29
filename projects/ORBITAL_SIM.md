# Orbital Sim

## Reasoning
This is a project I've selected to develop in order to expand my horizions and overall domain in simulation development. Additionally it is to refine my capability with threading and hpc development. Where this will not be a direct hpc implementation, I will be building from hpc principles. Examples of this being: tasks higly limited shared resources, limiting stealing due to latency of communication between nodes, dynamic load balancing via worker stealing, and SIMD register packing with type pruning. Where type pruning may not be the best for all hpc systems due to the adopting of ai-focused hpc hardware optimizing for floats. 

**KDK Simplification**
Due to the nature of pm simulation, leapfrog can be simplified by setting "initial" conditions to start at a half-step rather than at t = 0. From that point, you can update v using a single operation full-step. The reason this is allowed is because kdk looks like this:
kick(1/2 step velocity) drift(move) kick(now at full step), kick(1/2 step)...
Because kick at full step andkick and 1/2 step in between cycles happen at the same position, v will always be the same for those two steps, so it can be simplified into:

kick(1/2 step velocity), drift(move) kick(now at next 1/2 step)

- FFTW3f takes control of threading for the sake of limiting implementation complexity, so the scheduler must wait until it's completed before moving to step 3. Since you can optionally control threading with fftw3f, this will need to be replaced later for an hpc compatible fft 

- There will be two primary coordinate systems:
    1. Real Space: this is the coordinate system for particles, these are a dynamic range of values
    2. Grid Space: a fixed coordinate system for the actual grid used.
The reason being, a dynamic grid requires significantly more resources and is less cache friendly. To translate between the two systems, there are running mins and maxes for particles in the simulation, and these are used to interpolate from real space to grid space. 

## Sketch:
- Orbital Sim: governor and orchestrator of the simulation steps
- MassMesh: A mesh for depositing mass to a grid
- AccelField: A mesh object that is translated from the poisson solve of the mass mesh
- Scheduler: A worker-stealing scheduler for dynamic load balancing. Cache-based for complex dynamic schedules, and non-cached when the schedules are simple and relatively static. Targeted caching like this is a balancing act of whether the ram needed to store the cache is more valuable than the speed of caching.
    - Tasks: Tasks are specifically built around operating on local-only resources. Since this will not be a direct hpc implementation due to lacking 
- Storage: Ecs-style with free-lists for fixed-index particles. More removal friendly than standard vectors, and index becomes fixed, so there is no need to id particles directly.
- vec3: A dedicated vec3 module that I can extend as-needed for whatever processes I need.

Project repository is available [here](https://github.com/AdamP1592/OrbitalSim).