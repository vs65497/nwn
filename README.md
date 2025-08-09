# **Reproducing Emergent Brain-Like Complexity in Nanowire Atomic Switch Networks**

**Reproduced from**: 

_Z. Kuncic et al., "[Emergent brain-like complexity from nanowire atomic switch networks: Towards neuromorphic synthetic intelligence](https://ieeexplore.ieee.org/document/8626236)," 2018 IEEE 18th International Conference on Nanotechnology (IEEE-NANO), Cork, Ireland, 2018, pp. 1-3, doi: 10.1109/NANO.2018.8626236._

## **1. Results**

| Network Topology | Input PWM Signal | Output Network Conductance |
|---------|---------|---------|
| ![Alt text 1](https://github.com/vs65497/nwn/blob/main/network.png) | ![Alt text 2](https://github.com/vs65497/nwn/blob/main/input.png) | ![Alt text 3](https://github.com/vs65497/nwn/blob/main/output.png) |

_Figure 1. Reproduced results. Left: Random network topology. Center: Input PWM signal. Right: Output network conductance._

### **Comparison to Original Paper**

* My results **qualitatively resemble** those in the original paper. The network exhibits **memory-like behavior**, with conductance persisting after pulse inputs stop.
* I did **not calculate power spectral density** or other advanced metrics from the original study because I believe that the network size must be larger and the timestep shorter, which are limited by the processing speed my Python implementation.
* I also ignored **quantized conductance** effects for simplicity.

In effect, I have accomplished objectives #1 and #2:
1. To reproduce the **network topology** as described in the original work.
2. To reproduce the conductance-based **response characteristics** of the network from PWM input.

### **Performance Observations**

* Conductance rises and remains elevated in response to PWM pulses, suggesting formation and retention of conductive paths.
* Early wire junctions ("neurons") respond almost instantaneously to input signals, indicating low latency and rapid bridge formation.
* Time resolution became a bottleneck: bridge dynamics occurred faster than my simulation timestep. This further justified moving to C++ for improved granularity.
* Lesson learned: for long-term simulation projects or networks with fast-changing dynamics, begin in C++ or a similarly performant environment.

### **Takeaways**
This study gave me more confidence to pursue nonlinear dynamical systems and chaos, complexity, and emergence in practice. I have particularly found conversations surrounding "emergence" as difficult because it is not easily demonstrated. John Conway's Game of Life (https://en.wikipedia.org/wiki/Conway's_Game_of_Life), for example, does do a great job at presenting emergence. But even he did not think anything can be done with his discovery, calling it "finished." (https://youtu.be/R9Plq-D1gEk?si=EYvjvpzMf5HmQZY3&t=458) 

Demonstrating memory in a disorganized system, on my local computer, proved to me that some of my intuitions may be right. At least I have seen that this is worth investigating further.

## **2. Context**

### **Introduction**

In a sequence of ideas, perhaps beginning with Turing's concept of unorganized computers (Turing, Intelligent Machinery, 1948), the field of neuromorphic computing proposes structures modeled after the brain. The term "neuromorphic" meaning brain-like with "neuro" coming from the Greek word for neuron, and "morphic" coming from the Greek word morphē, meaning “form” or “shape.” Given its name by Carver Mead in 1980, Mead makes the case that biological solutions for information-processing systems (I interpret this as meaning "intelligent" systems) surpass digital methods, due to "the use of elementary physical phenomena as computational primitives", and likewise methods using analog signals are superior to those using digital signals. He states that applying adaptive techniques to account for differences between analog and digital components will "naturally" lead to systems which "learn about their environment." (Mead, 1980)

Following this thinking into practice, we find physical implementations of neural networks. In this study, I explore how Kuncic et. al. simulate an atomic switch network. Each junction between silver-sulfide (Ag2S) nanowires act as a memristor which "mimics the chemical synapse between neurons in response to electrical stimuli," and "when connected together in a self-organized manner, similar to a neuronal network, atomic switch networks exhibit emergent brain-like complexity properties, including nonlinear stochastic dynamics and memorization, making them a unique experimental system for emulating intelligence." (Kuncic et. al., 2018)

### **Motivation for Reproduction**

My interest in this work is the result of an apparent "wandering" between many fields; some of which is visible in my portfolio. This wandering does not come from a lack of focus. Instead, my behavior is the result of chasing some possible system capable of online, adaptive learning. Through interacting with the material (mentioned in the Influences section), I have been slowly approaching the principles found in Reservoir Computing. Professor Kuncic's work was the first I've encountered that expresses some of these ideas. And this makes it the ideal candidate for deep study -- reproduction.

### **Objectives of the Reproduction**

Given my interest in Professor Kuncic's paper, I sought to gain a deeper understanding of the ideas presented by reproducing three key aspects of Kuncic's 2018 paper:

1. The **network topology** as described in the original work.
2. The conductance-based **response characteristics** of the network from PWM input.
3. The **relationship between frequency and conductance behavior**.

Provided the topology was correctly established, and the neurodal behavior was correctly modeled, showing temporal output gives evidence that the overall silver-sulfied nanowire network (Ag2S-NWN) can produce an emergent singal local rules. 

## **3. Methods**

### **Original Paper’s Methodology**

The original paper describes a computational model built in MATLAB to simulate Ag2S-NWNs:

* Wires are randomly distributed in 2D space with positions and orientations sampled uniformly, and lengths sampled from a gamma distribution.
* Intersections between wires create junctions where atomic switches form. Each switch is modeled as a function of time and local voltage, with conductive bridge growth occurring when the voltage exceeds a threshold.
* Once the bridge exceeds a critical length, the switch "turns on," dropping resistance by at least two orders of magnitude.
* The system is abstracted as a graph: wires become vertices, junctions become edges. The resulting adjacency matrix and Kirchhoff’s laws allow simulation of current flow across the network, effectively reproducing experimental probe measurements.

### **My Implementation**

1. To solve voltages, I first convert the network to it's dual representation, with wires becoming nodes and bridge junctions becoming edges (_see Figure 2_). Each edge carries a resistance initialized to the OFF state (non-zero). Then I view each wire (now a node) as its independent connections (_see Figure 3_). And finally, I generalize the wire view to its general topology (_see Figure 4_). This allows for homologous KCL nodal equations.
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

### **Critical Gaps and Problem-Solving**

* The original paper states only that node voltages were solved simultaneously using Kirchhoff’s Current Law (KCL). While one sentence may be sufficient in hindsight, arriving at that conclusion took considerable effort. I experimented with several alternative approaches before realizing that simultaneous solving was likely the only robust method for a random, highly connected graph.

* The challenge stems from the network’s complexity: any node can branch into an arbitrary number of subgraphs, and these branches may loop back on themselves rather than progress toward the low-voltage terminal. This recurrence strongly resembles the feedback structure of Recurrent Neural Networks (RNNs) and helps explain why methods like Backpropagation Through Time (BPTT) are nontrivial. From this perspective, it was satisfying to observe the “weights” (conductances) of each junction naturally self-organize in response to the input waveform.

* Another insight emerged from the fact that, without quantized conductance, the network behaves more like a discrete system than a fully continuous one. Even with quantization, the conductance output is unlikely to be perfectly smooth. This suggests that the network exhibits distinct operational modes, analogous to harmonic resonances. I suspect this may relate to the “edge of chaos” phenomenon—where information organizes into coherent structures under just the right conditions—and believe it merits further investigation.

* Wire currents don't follow the rules of circuit theory because they are nodes in the network dual representation. There is likely a much better way to model current directions in wires by accounting for micro or nano-Ohm resistances per length of wire. In general, however, drawing currents is likely better for visualization than for quantitative value. Despite any faulty edge cases, I think this implementation is sufficient in understanding network dynamics.

## **4. Additional Information**

### **Future Directions**

* Continue building on Kuncic’s line of research to further explore the neuromorphic potential of nanowire networks.
* Model bridge dynamics with quantized conductance to better match the behavior of physical devices.
* Investigate optimal network topologies, such as those used in Echo State Networks (ESNs), and assess their advantages over purely random structures.
* Evaluate randomized networks for the Echo State Property (ESP) to determine their suitability for reservoir computing.
* Develop alternative methods for designing topology in simulation before construction, potentially using approaches like magnetic field-guided assembly instead of random initialization.
* Explore ways to compensate for fixed physical topologies through feedback and feedforward mechanisms.
* Implement verification methods to quantitatively link individual bridge states to network-level conductance, analyze the impact of graph structure on conductance, and measure the nonlinearity of these relationships—addressing the current lack of confirmation that simulated or reported networks behave like true NWNs.
* Investigate whether modeling the network in two dimensions significantly affects performance compared to a three-dimensional architecture.
* Study the precise relationship between PWM inputs and memory retention, and test how the network responds to irregular, information-rich signals such as speech patterns.

### **Implementation Lessons for Future Work**

* **Bridge modeling** should incorporate quantized conductance to reflect real-world behavior more accurately.
* **Simulation timestep** must be selected based on bridge growth rates to avoid skipping critical dynamics.
* Consider integrating GPU acceleration or sparse solvers for real-time or large-scale simulations.

### **Influences**:

* **Underactuated robotics (Russ Tedrake)** (https://underactuated.csail.mit.edu/): Underactuated control means controlling a robots actions around degrees of freedom for which you don't have actuators. This is significant because it is a first step towards exploiting enviornmental dynamics rather than modeling and controlling them (which is how biological organisms deal with the world). 
* **Control theory vs. machine learning**: This is effectively discussing the difference between modeling from first-principles versus modeling from data. The tension between these fields has led me to ponder the true representation of phenomena and information (in the abstract sense), what it means to "predict" events or to "understand" ideas, and what happens when we use our intellect to accomplish goals in the world.
* **Swarm Intelligence: From Natural to Artificial Systems (Bonabeau, Dorigo, Theraulaz)** (https://academic.oup.com/book/40811): Swarms are collections of entities which utilize a decentralized means of making decisions and performing actions. A section in the book discusses "stygmergy" where bees, for example, additively build their nests block-by-block rather than following an overall schema. The result is that some of the computation for the building is outsourced from the bee to the structure. This points to the idea of using matter to directly compute rather than logic. At the time, this caused me to look for analog computers.
* **Neurorobotics (Tiffany Hwu, Jeff Krichmar)** (https://mitpress.mit.edu/9780262047067/neurorobotics/): I mainly drew three ideas from this book: embodiment can give robots natural behavior without modeling and without data, rich environments can provide robots the ability to learn more complex behaviors, and rich bodies are important for taking advantage of such rich environments. In a somewhat indirect way, this eventually led me to developmental robotics and to the idea of bowties (https://en.wikipedia.org/wiki/Bow_tie_(biology)), to which RC's have some relation.
* **Complexity (Melanie Mitchell)** (https://a.co/d/33P8S10): "Complexity" motivated me to investigate bottom-up systems, self-organization/assembly, and emergent behavior.
* **"A robust layered control system for a mobile robot" (Rodney Brooks)** (https://ieeexplore.ieee.org/document/1087032): Brooks' subsumption architecture reinforced my interest in behavior-based robotics and low-level intelligent architectures.
* **Synthetic Intelligence (Zdenka Kuncic)** (https://www.youtube.com/watch?v=c3EVUogQr6k): Professor Kuncic's presentation on nanowire networks and "silver neurons" intrigued me. Although I saw this before everything else, it wasn't until after I had interacted with all of the above influences that I returned to this lecture with seriousness.
