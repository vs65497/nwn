# **Reproducing Emergent Brain-Like Complexity in Nanowire Atomic Switch Networks**

**Reproduced from**: 

_Z. Kuncic et al., "[Emergent brain-like complexity from nanowire atomic switch networks: Towards neuromorphic synthetic intelligence](https://ieeexplore.ieee.org/document/8626236)," 2018 IEEE 18th International Conference on Nanotechnology (IEEE-NANO), Cork, Ireland, 2018, pp. 1-3, doi: 10.1109/NANO.2018.8626236._

<br>

## **1. Overview**

| Network Topology | Input PWM Signal | Network Output Conductance |
|---------|---------|---------|
| ![Network Topology](https://github.com/vs65497/nwn/blob/main/network.png) | ![Input PWM Signal](https://github.com/vs65497/nwn/blob/main/input.png) | ![Network Output Conductance](https://github.com/vs65497/nwn/blob/main/output.png) |

_Figure 1. Reproduced results. Left: Random network topology. Center: Input PWM signal [0.01, 5.00] Volts. Right: Network Output Conductance._

<br>

### **Purpose** ([_read more_](https://github.com/vs65497/nwn?tab=readme-ov-file#2-purpose))

  * To reproduce the "memory"-like behavior, indicated by output conductance, of a simulated silver-sulfide nanowire network (Ag2S-NWN), as seen in _Kuncic 2018_. The output conductance is observed having a steep rise while an input PWM signal is enabled, then settles at a non-zero value after the input is disabled.

### **Results**

  1. Successfully reproduced the topology of a simulated Ag2S-NWN (_see Figure 1, Network Topology_).
  2. Reproduced the output conductance characteristics, with "memory"-like behavior, of the network (total conductance of all junctions versus time) by using a PWM input signal (_see Figure 1, Network Output Conductance_). Please see the [_Discussion_](https://github.com/vs65497/nwn?tab=readme-ov-file#4-discussion) section for more details.

### **Methods** ([_read more_](https://github.com/vs65497/nwn?tab=readme-ov-file#3-methods))

  * To calculate network voltages, I simultaneously solved Kirchhoff’s Current Law (KCL) equations for all nodes in the network at each timestep. Bridge growth, annihilation, and conductance were calculated based on a model found in the paper _T. Hasegawa, 2011_.
  * _T. Hasegawa et al., "[Atomic Switch: Atom/Ion Movement Controlled Devices for Beyond Von-Neumann Computers](https://advanced.onlinelibrary.wiley.com/doi/10.1002/adma.201102597)," 2011 Wiley Advanced Materials, doi: 10.1002/adma.201102597._

### **Discussion** ([_read more_](https://github.com/vs65497/nwn?tab=readme-ov-file#4-discussion))

  * Conductance rises due to input PWM and remains elevated after disabling input signal, suggesting formation and retention of conductive paths.
  * Early wire junctions ("neurodes") respond almost instantaneously to input signals, indicating low latency and rapid bridge formation.
  * Low amount of samples per time (low-resolution) became a bottleneck: bridge dynamics occurred faster than my simulation timestep.

<br><br>

## **2. Purpose**

### **Introduction**

In a sequence of ideas, perhaps beginning with Turing's concept of unorganized computers (Turing, Intelligent Machinery, 1948), the field of neuromorphic computing proposes structures modeled after the brain. The term "neuromorphic" meaning brain-like with "neuro" coming from the Greek word for neuron, and "morphic" coming from the Greek word morphē, meaning “form” or “shape.” Given its name by Carver Mead in 1980, Mead makes the case that biological solutions for information-processing systems (I interpret this as meaning "intelligent" systems) surpass digital methods, due to "the use of elementary physical phenomena as computational primitives", and likewise methods using analog signals are superior to those using digital signals. He states that applying adaptive techniques to account for differences between analog and digital components will "naturally" lead to systems which "learn about their environment." (Mead, 1980)

Following this thinking into practice, we find physical implementations of neural networks. In this study, I explore how Kuncic et. al. simulate an atomic switch network. Each junction between silver-sulfide (Ag2S) nanowires act as a memristor which "mimics the chemical synapse between neurons in response to electrical stimuli," and "when connected together in a self-organized manner, similar to a neuronal network, atomic switch networks exhibit emergent brain-like complexity properties, including nonlinear stochastic dynamics and memorization, making them a unique experimental system for emulating intelligence." (_Kuncic 2018_)

### **Objectives**

1. To reproduce the **network topology** as described in the original work.
2. To reproduce the conductance-based **response characteristics** of the network from PWM input. The output conductance should remain high after disabling the input signal.
3. To reproduce the **relationship between frequency and conductance behavior**.

## **3. Methods**

### **Kuncic 2018 Methodology**

The original paper describes a computational model built in MATLAB to simulate Ag2S-NWNs:

* Wires are randomly distributed in 2D space with positions and orientations sampled uniformly, and lengths sampled from a gamma distribution.
* Intersections between wires create junctions where atomic switches form. Each switch is modeled as a function of time and local voltage, with conductive bridge growth occurring when the voltage exceeds a threshold.
* Once the bridge exceeds a critical length, the switch "turns on," dropping resistance by at least two orders of magnitude.
* The system is abstracted as a graph: wires become vertices, junctions become edges. The resulting adjacency matrix and Kirchhoff’s laws allow simulation of current flow across the network, effectively reproducing experimental probe measurements.

### **My Implementation**

1. To solve voltages, I first convert the network to its dual representation, with wires becoming nodes and bridge junctions becoming edges (_see Figure 2_). Each edge carries a resistance initialized to the OFF state (non-zero). Then I view each wire (now a node) as its independent connections (_see Figure 3_). And finally, I generalize the wire view to its general topology (_see Figure 4_). This allows for homologous KCL nodal equations.
2. With all wires in their general form, I now apply KCL to the entire network. By separating out wires with known voltages (Vcc and GND), we can then move these known current values to the RHS and apply the Moore-Penrose Pseudo Inverse to simultaneously solve for network voltages (_see Figure 5_).
3. Lastly, currents are computed from voltage differences across branches (_see Figures 6 and 7_). Directionality arises from current imbalances (e.g., more current entering from one side of a node).

    <img src="https://github.com/vs65497/nwn/blob/main/figure2.png" width="auto">
    
    _Figure 2. Network conversion to the dual representation._
   
    <img src="https://github.com/vs65497/nwn/blob/main/figure3.png" width="auto">
    
    _Figure 3. Dual representation separated to wire (nodal) view._
   
    <img src="https://github.com/vs65497/nwn/blob/main/figure4.png" width="auto">
    
    _Figure 4. Generalization of all possible wire topologies._
   
    <img src="https://github.com/vs65497/nwn/blob/main/circuit_solver.png" width="auto">
    
    _Figure 5. Simultaneously solving for network voltages._
   
    <img src="https://github.com/vs65497/nwn/blob/main/figure6.png" width="auto">
    
    _Figure 6. Modified general wire topology, also including wire ends._
   
    <img src="https://github.com/vs65497/nwn/blob/main/current_solver.png" height="auto">
    
    _Figure 7. Solving for all wire currents._

## **4. Discussion**

### **Observations**

* Any section of the network can take the form of an arbitrary subgraphs of arbitrary direction. Walks of these subgraphs which loop back onto themselves show recurrence in the network -- this may partly explain why the network holds memory. When combined with the growth/annihilation of bridges, there appears to be a complex movement of charges through the network, thus describing rich dynamics.

* The network responds with an output conductance in discrete steps. This may have to do with the small amount of neurodes or a lack of implementation of quantized conductance. It may be possible that the output conductance approaches a smooth curve (differentiable) with quantized conductance.

* Circuit theory only describes current in branches, not in nodes. In our network, wires behave as nodes with distance between branches. Regardless if current is defined as the movement of electrons or charges, a current will between branches on the same wire. At least when using our idealized depiction of the network, we cannot use Ohm's Law to determine the direction of the currents because the wires are ideal conductors (zero resistance). Perhaps a better way of tracking the direction of the in-wire current is to use a measure of per-length resistivity. However, painting currents on the canvas is likely better for visualization than for analytical value. There is at least one faulty edge case in the implementation of my current painting function, but this is likely sufficient to understand the flow of charge in the network per an arbitrary snapshot.

### **Future Directions**

* Study the relationship between PWM inputs and memory retention. Test how the network responds to irregular, information-rich signals such as speech patterns.
* Investigate network topologies, such as Echo State Networks (ESNs) and Small-World Networks (SWNs), and assess their advantages over purely random structures. Perform an ablation study to determine the effect of network configuration on the its performance.
* ESNs -- Evaluate the network from the lens of the Echo State Property (ESP). Because all neurodes are either on or off, the SVD may need to be determined using the total resistance of each wire and its common neurodes rather than each neurode individually.
* Model bridge dynamics with quantized conductance to better match the behavior of physical devices.

### **Influences**:

  1. **Underactuated robotics (Russ Tedrake)** (https://underactuated.csail.mit.edu/): Underactuated control means controlling a robots actions around degrees of freedom for which you don't have actuators. This is significant because it is a first step towards exploiting enviornmental dynamics rather than modeling and controlling them (which is how biological organisms deal with the world). 
  2. **Control theory vs. machine learning**: This is effectively discussing the difference between modeling from first-principles versus modeling from data. The tension between these fields has led me to ponder the true representation of phenomena and information (in the abstract sense), what it means to "predict" events or to "understand" ideas, and what happens when we use our intellect to accomplish goals in the world.
  3. **Swarm Intelligence: From Natural to Artificial Systems (Bonabeau, Dorigo, Theraulaz)** (https://academic.oup.com/book/40811): Swarms are collections of entities which utilize a decentralized means of making decisions and performing actions. A section in the book discusses "stigmergy" where bees, for example, additively build their nests block-by-block rather than following an overall schema. The result is that some of the computation for the building is outsourced from the bee to the structure. This points to the idea of using matter to directly compute rather than logic. At the time, this caused me to look for analog computers.
  4. **Neurorobotics (Tiffany Hwu, Jeff Krichmar)** (https://mitpress.mit.edu/9780262047067/neurorobotics/): I mainly drew three ideas from this book: embodiment can give robots natural behavior without modeling and without data, rich environments can provide robots the ability to learn more complex behaviors, and rich bodies are important for taking advantage of such rich environments. In a somewhat indirect way, this eventually led me to developmental robotics and to the idea of bowties (https://en.wikipedia.org/wiki/Bow_tie_(biology)), to which RC's have some relation.
  5. **Complexity (Melanie Mitchell)** (https://a.co/d/33P8S10): "Complexity" motivated me to investigate bottom-up systems, self-organization/assembly, and emergent behavior.
  6. **"A robust layered control system for a mobile robot" (Rodney Brooks)** (https://ieeexplore.ieee.org/document/1087032): Brooks' subsumption architecture reinforced my interest in behavior-based robotics and low-level intelligent architectures.
  7. **Synthetic Intelligence (Zdenka Kuncic)** (https://www.youtube.com/watch?v=c3EVUogQr6k): Professor Kuncic's presentation on nanowire networks and "silver neurons" intrigued me. Although I saw this before everything else, it wasn't until after I had interacted with all of the above influences that I returned to this lecture with seriousness.
