# The Rise and Fall of Maritime Empires

### An Agent-Based Model of Economic Inequality and Imperial Collapse

**Course:** Agent-based modelling for archaeologists
**Date:** February 2026
**Creator:** Orfeas Dialinos

## 1. Introduction & Overview

### 1.1 Purpose

This model extends the standard "Pond Trade" concept by [Andreas Angourakis]([url](https://github.com/CoDArchLab-ABM/course-guide/)) to simulate how geography, economic feedback loops and political instability contribute to the rise of empires and their eventual collapse. The simulation aims to demonstrate the "Rich Get Richer" phenomenon and the "Crisis of the Third Century" scenario where trade networks disintegrate into total war.

### 1.2 Entities, State Variables, and Scales

* **Patches:** Represent land (green) or water (blue). Water patches have a movement cost of 1; land patches are obstacles (infinite cost).
* **Settlements:** Static agents representing coastal cities. They possess variables for:
* `Size`: Population and economic power.
* `Stock`: Tradeable goods (wealth).
* `Culture`: A vector of traits (color, aggression, production).


* **Traders:** Mobile agents representing fleets. They possess:
* `Cargo`: Goods carried between ports.
* `Route`: A specific A* calculated path.
* `Home Base`: The city they belong to.



### 1.3 Process Overview and Scheduling

At every time step (tick), the following sequence occurs:

1. **Settlements** produce stock and consume resources based on their size.
2. **Settlements** spawn traders if their economy allows it.
3. **Traders** navigate using A* pathfinding to reach a target city.
4. **Interaction:** Upon arrival, the trader interacts based on the global `Imperial Stability` parameter:
* **High Stability:** The trader exchanges goods (Trade), increasing both cities' wealth.
* **Low Stability:** If the cultures are different, the trader raids the city, destroying population and stealing stock (War).


5. **Global Data:** The model calculates the Gini coefficient and updates the Lorenz Curve to visualize inequality.

---

## 2. Design Concepts

* **Emergence:** Economic inequality is not hard-coded but emerges from geographical advantages. Cities on central trade routes naturally grow faster, attracting more trade in a positive feedback loop.
* **Adaptation:** Agents do not move randomly; they adapt to the terrain using the A* algorithm, finding the most efficient maritime routes around procedurally generated continents.
* **Interaction:** The relationship between agents shifts from cooperative (Trade) to antagonistic (War) based on a threshold of "Cultural Distance" and global stability.
* **Stochasticity:** Map generation, cultural mutation, and initial settlement placement are randomized to ensure no two simulations are identical.

---

## 3. Development Process & "Dev Log"

This project evolved through several iterations of failure, debugging, and refinement.

### Phase 1: The "Ghost Ship" Failure

* **The Challenge:** Initially, I attempted to use standard NetLogo movement (`face target`, `forward 1`).
* **The Fail:** Agents ignored geography, walking "through" continents and walls. The simulation lacked physical realism.
* **The Fix:** I implemented a Grid-Based A* Pathfinding algorithm. This forces agents to calculate valid water paths before moving.

### Phase 2: The "Godzilla" Stalemate

* **The Challenge:** When implementing the War mechanic, I initially used a realistic damage formula where defenses mitigated attacks.
* **The Fail:** Large cities became indestructible. They regenerated population faster than enemies could raid them, leading to an infinite stalemate with no "Winner."
* **The Fix:** I implemented a "Nuclear" or "Total War" logic. Defense was removed, and damage was set to a 1:1 ratio (1 cargo unit kills 1 population unit). This allowed for decisive victories and the total collapse of weaker states.

### Phase 3: The Data Visualization

* **The Challenge:** It was difficult to visually confirm if "The Rich were getting Richer" just by looking at map sizes.
* **The Fix:** I added a real-time Lorenz Curve plot. This mathematically proved the transition from Equality (Diagonal line) to Oligarchy (Boomerang curve).

---

## 4. Verification & Unit Testing

To ensure the model behaved realistically and the code was robust, I conducted a series of isolated "Unit Tests" during each phase of development.

### 4.1 Navigation & Physics Tests (The A* Algorithm)

**Test A: The "Great Wall" Test (Pathfinding)**

* **Hypothesis:** Agents should recognize land as an obstacle and calculate a path around it.
* **Procedure:** I manually constructed a vertical barrier of land patches cutting the central ocean in half. I then commanded agents to travel from the far left to the far right.
* **Result (Success):** Agents successfully calculated a U-shaped path, sailing North or South around the tips of the wall rather than attempting to walk through it. This confirmed the A* algorithm was correctly reading the `is-land` boolean.

**Test B: The "Strait of Gibraltar" Test (Choke Points)**

* **Hypothesis:** Agents should prioritize valid water paths, even if they are narrow and indirect.
* **Procedure:** I created a solid wall blocking the map but deleted a single patch in the center to create a 1-pixel "strait."
* **Result (Success):** Agents from all coordinates funneled specifically through that single pixel gap. This confirmed the heuristic function was correctly weighting movement costs.

**Test C: The "Pen Trick" Visualization**

* **Hypothesis:** Shipping lanes should look organic, not robotic.
* **Procedure:** I executed the command `ask traders [ pen-down ]` and let the simulation run for 500 ticks.
* **Result (Success):** The traders drew permanent lines on the map. The resulting drawing showed organic shipping lanes curving around coastlines, visually resembling real-world maritime traffic maps.

### 4.2 Mechanic Verification (War & Trade)

**Test D: The "Red Flash" Trigger**

* **Hypothesis:** The transition from Trade to War should happen instantly when stability drops.
* **Procedure:** I set `imperial-stability` to 100, observed peaceful movement, and then dragged the slider to 0 in real-time.
* **Result (Success):** Settlements immediately flashed **Red** (visual feedback), confirming that the `rome-unload-cargo` procedure was correctly switching branches from "Trade" to "Raid" based on the slider value.

**Test E: The Stalemate (Parameter Tuning)**

* **Hypothesis:** War should lead to the collapse of weaker states.
* **Initial Result (Failure):** When running the war scenario, the wealth graph remained static.
* **Diagnosis:** The damage formula (`Damage / 100`) was too lenient. Large cities (Size > 1000) regenerated population naturally faster than the enemy could destroy it.
* **Correction:** I removed the defense mitigation and set the damage ratio to 1:1 (`Damage = Population Loss`). This "Nuclear Option" ensured that attacks had permanent, strategic consequences.

### 4.3 Simulation Outcome Tests (Data Analysis)

**Test F: The "Equality Baseline" Calibration**

* **Hypothesis:** To prove the Lorenz curve works, it must show "Perfect Equality" when forced.
* **Procedure:** I injected a command `ask settlements [ set sizeLevel 50 ]` to force all cities to be identical.
* **Result (Success):** The Lorenz curve graph snapped to the diagonal gray line (45 degrees), confirming the plotting math was accurate.

**Test G: The "Winner Takes All" Outcome**

* **Hypothesis:** In a low-stability environment, the simulation should end with a Monopoly or Duopoly.
* **Procedure:** I ran the model with the corrected damage logic at Stability 0 until activity ceased. I then ran the command `show reverse sort [sizeLevel] of settlements`.
* **Final Result (Success):** The output consistently showed a distribution resembling `[ 2300, 150, 1, 1, 1, 1 ]`. This mathematically proved that the model successfully simulates the collapse of competitive markets into imperial dominance.

**Test H: Turbo Mode Performance**

* **Hypothesis:** The simulation must be fast enough to simulate centuries of history.
* **Procedure:** I implemented a view update throttle (`if ticks mod 50 = 0`).
* **Result (Success):** The simulation speed increased by ~50x, allowing 20,000 ticks (an "Era") to complete in under 120 seconds.

---

## 5. Technical Implementation Details

### 5.1 The A* (A-Star) Pathfinder

To solve the problem of agents walking through continents, I implemented the A* algorithm. This is a best-first search algorithm that finds the least-cost path from a given initial node to one goal node.

* **Heuristic (h):** I used Euclidean distance (`distance end-patch`) as the heuristic. This allows the agent to guess which patches are likely to lead to the target.
* **Cost Function (g):**
* Water patches have a movement cost of `1`.
* Land patches have an infinite cost (or `999`), effectively acting as walls.


* **Optimization:** Instead of recalculating the path every tick (which would freeze the simulation), agents calculate the route **once** upon spawning. They store the list of patches in a `route` list and simply traverse it index by index.

### 5.2 The "War" Logic (The Crisis Mechanic)

The transition from Trade to War is governed by a **Threshold Function**.

`Threshold = Imperial Stability * 4`

* **Logic:**
1. The agent calculates the Euclidean distance between its `Cultural Vector` (RGB color) and the target settlement's vector.
2. If `Cultural Distance > Threshold`, the interaction is flagged as **Hostile**.
3. Because `Imperial Stability` is controlled by a slider (0-100), the user can dynamically shrink the threshold. At 0 stability, the threshold is 0, meaning *any* difference in culture triggers a war.



### 5.3 The Inequality Engine (Lorenz Curve)

To visualize the "Wealth of Nations," the model sorts all settlements by wealth (Size + Stock) and plots the cumulative distribution.

* **X-Axis:** Cumulative % of Population (sorted from poorest to richest).
* **Y-Axis:** Cumulative % of Wealth held.
* **Interpretation:**
* A straight 45-degree line indicates strict equality (10% of people own 10% of wealth).
* A curve dipping below the line indicates inequality.
* The "War" phase causes this curve to snap to the bottom-right, mathematically demonstrating that conflict accelerates wealth concentration.



---

## 6. AI Assistance Acknowledgement

In accordance with course guidelines regarding the use of Large Language Models (LLMs), I declare the following assistance in this project:

**Primary Tool:** Google Gemini (Thought Partner)

**Scope of Usage:**

1. **Troubleshooting A* Algorithm:** Writing a custom A* pathfinder in NetLogo is syntactically complex. I used AI to generate the boilerplate code for the `open-list` and `closed-list` logic, which I then integrated and tuned for my specific variable names.
2. **Logic Refinement:** When the War simulation resulted in a stalemate, I consulted the AI to analyze the mathematical flaw in my damage formula. The AI suggested the "1:1 damage ratio" to break the deadlock.
3. **Data Visualization:** The code for drawing the dynamic Lorenz Curve (calculating the cumulative sum of wealth) was co-written with AI assistance to ensure performance efficiency in "Turbo Mode."

**Statement of Integrity:**
While code snippets were generated with assistance, the conceptual modeling, parameter tuning, testing, and final integration were performed by me. The decision to switch from a trade model to a collapse model was my own design choice.

---

## 7. User Manual / How to Run

### Step 1: Setup (The Golden Age)

1. Open `Rome_Empire_Simulation.nlogo`.
2. Set `imperial-stability` to **100**.
3. Set `pondSize` to **20** and `numberOfSettlements` to **15**.
4. Press **Setup** and then **Go**.
5. *Observation:* Watch as cities grow large and the "Wealth Distribution" graph shows a curve indicating natural market inequality.

### Step 2: The Crisis (The Collapse)

1. Once several cities have reached Size > 50 (labels appear), drag the `imperial-stability` slider to **0**.
2. *Observation:* Settlements will flash RED. The population numbers will crash. The Wealth Graph will sag deeply into the bottom-right corner.

### Step 3: The End State

1. Allow the model to run until flashing stops.
2. Open the Command Center and type: `show reverse sort [sizeLevel] of settlements`
3. *Result:* You will see 1 or 2 massive numbers (the Winners) and a list of 1s (the destroyed civilizations).

---

## 8. References

1. **Original Model:** "Pond Trade" (NetLogo Models Library).
2. **Algorithm:** Hart, P. E., Nilsson, N. J., & Raphael, B. (1968). "A Formal Basis for the Heuristic Determination of Minimum Cost Paths". *IEEE Transactions on Systems Science and Cybernetics*.
3. **Theory:** Turchin, P. (2003). *Historical Dynamics: Why States Rise and Fall*. Princeton University Press. (Used for the concept of imperial stability).
4. **Methodology:** Grimm, V., et al. (2006). "A standard protocol for describing individual-based and agent-based models". *Ecological Modelling*.
