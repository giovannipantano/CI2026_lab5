# Wumpus World: a hard-coded policy and a learned one

Computational Intelligence · Politecnico di Torino · Giovannni Pantano (s362243)

## 🎥 Video presentation

**[Watch the video on Google Drive](https://drive.google.com/file/d/1rfFpbhr_wItYziyeEdaEdLmFsQ0xKGYm/view?usp=sharing)**

## The problem

An agent in a 4×4 cave must grab the gold and climb out from square (1,1). The cave holds the Wumpus at (1,3), the gold at (2,3) and pits at (3,1), (3,3) and (4,4).

| Event | Reward |
|---|---|
| climb out with the gold | +1000 |
| die (pit or Wumpus) | −1000 |
| every action | −1 |
| shoot the arrow | −10 |

The agent **never perceives its position**: at each step it only receives sensor messages (`breeze`, `stench`, `glitter`, `bump`, `scream`, `at_start`). Since it knows its own moves, it keeps track of its position, direction and whether it carries the gold, and adds these as internal messages (`x2`, `y1`, `facing_north`, `has_gold`/`no_gold`). Sensors + memory form the **agent state**, which contains everything needed to decide.

## Solution 1: a hard-coded policy

[`wumpus_1_hardcoded.ipynb`](wumpus_1_hardcoded.ipynb)

The policy is an ordered list of 10 rules `condition → action`, written by hand. A rule matches when its condition is a subset of the current messages, and the agent fires the **first rule that matches**, so special cases come before general ones:

```python
rule("glitter no_gold",         "grab"),    # see the gold: take it
rule("has_gold at_start",       "climb"),   # home with the gold: leave
rule("no_gold x2 facing_north", "fwd"),     # one rule covers all of column 2
...
```

## Solution 2: a learned policy (Learning Classifier System)

[`wumpus_2_learned.ipynb`](wumpus_2_learned.ipynb)

The agent starts with **no rules** and doesn't know the map. It learns from the rewards with a Learning Classifier System. Each classifier is `condition → action` plus a **strength**.

| Component | Here |
|---|---|
| Performance | the matching rules bid `c · specificity · strength`; the highest bid wins (sometimes a random rule: exploration, fading to 0) |
| Reinforcement | Holland's **bucket brigade**: the winner pays its bid to the previous winner and receives the reward of its action |
| Discovery | **covering**: a missing rule is created, with an optimistic strength of 1000 |

## Results

| | Solution 1: hard-coded | Solution 2: learned |
|---|---|---|
| score in the cave from the slides | **988** (12 actions, the minimum) | **986** (14 actions) |
| training | none | 2000 episodes; first win at episode 69, 200/200 wins in the last 200 |
| gold moved to (4,2) | −60: the rules walk into a wall | 985: the same code learns a new route |
| rules | 10, some general | 14 used out of 702 created, all fully specific |

Other findings from solution 2:
- **8 random seeds:** every run wins, using 12 to 17 actions. The bucket brigade finds a good route, not always the shortest.
