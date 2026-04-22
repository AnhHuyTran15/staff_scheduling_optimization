# staff_scheduling_optimization
### 2.4 Objective Function
The objective utilizes Goal Programming with Penalty Costs. It minimizes the gap between the maximum and minimum shifts worked (Load Balancing), while heavily penalizing the assignment of full weekends to workers:

$$
\min \left( W_{max} - W_{min} \right) + 5 \sum_{i \in I} y_i
$$

*(Note: The weight of 5 indicates that avoiding a full weekend is prioritized over a single-shift imbalance between workers).*

### 2.5 Constraints

**Base Labor Constraints:**
- **Demand Satisfaction:** $\sum_{i \in I} x_{ijk} = d_{jk} \quad \forall j \in J, \forall k \in K$
- **Daily Workload:** Max 1 shift per day. $\sum_{k \in K} x_{ijk} \le 1 \quad \forall i \in I, \forall j \in J$
- **Weekly Workload:** Max 5 working days per week. $\sum_{j \in J} \sum_{k \in K} x_{ijk} \le 5 \quad \forall i \in I$
- **Rest-Period Constraint:** No Evening-to-Morning consecutive shifts. $x_{ij3} + x_{i(j+1)1} \le 1 \quad \forall i \in I, j \in \{1, \dots, 6\}$

**Advanced Optimization Constraints:**
- **Load Balancing Definition:** $\sum_{j \in J} \sum_{k \in K} x_{ijk} \le W_{max} \quad \forall i \in I$ and $\sum_{j \in J} \sum_{k \in K} x_{ijk} \ge W_{min} \quad \forall i \in I$
- **Skill Mix Requirement:** At least 1 Senior per shift. $\sum_{i \in S} x_{ijk} \ge 1 \quad \forall j \in J, \forall k \in K$
- **Weekend Overlap Identifier:** Forces $y_i$ to 1 if worker $i$ works at least one shift on both day 6 and day 7. $\sum_{k \in K} x_{i6k} + \sum_{k \in K} x_{i7k} - 1 \le y_i \quad \forall i \in I$
