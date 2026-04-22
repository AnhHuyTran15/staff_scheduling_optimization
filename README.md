# staff_scheduling_optimization
# Advanced Factory Staff Assignment Optimization

This project implements an advanced Mixed-Integer Linear Programming (MILP) model to optimize weekly staff assignments in a three-shift manufacturing environment. Using Python and the Gurobi optimization solver, the model goes beyond basic feasibility to ensure labor compliance, workload fairness, operational safety (skill mixing), and employee satisfaction regarding weekend shifts.

## 1. Problem Description

We consider a factory operating 7 days per week with 3 shifts per day (Morning, Afternoon, Evening). For each day–shift pair, a required number of workers is specified. The total workforce consists of 20 workers, subdivided into 5 Seniors and 15 Juniors. The goal is to generate an optimal weekly roster.

## 2. Business Rules & Optimizations

The model respects standard labor regulations (Hard Constraints) while optimizing for operational efficiency and worker well-being (Soft Constraints/Objectives).

**Standard Labor Rules:**
* A worker can work at most one shift per day.
* Each worker must have at least 2 leave days per week (i.e., at most 5 working days).
* **Rest Period:** If a worker works the Evening shift on a given day, they cannot work the Morning shift on the immediate next day.

**Advanced Optimizations:**
* **Skill Mix:** Every shift must have at least one Senior worker assigned to guide Juniors and handle complex issues.
* **Load Balancing:** The workload (total shifts worked) should be distributed as evenly as possible among all 20 workers.
* **Weekend Rest:** Working both Saturday and Sunday is highly undesirable. The model strictly minimizes the number of employees forced to work the full weekend.

## 3. Mathematical Formulation

### 3.1 Sets and Indices
* $I = \{1, 2, \dots, 20\}$: Set of all workers.
* $S = \{1, 2, 3, 4, 5\}$: Subset of Senior workers ($S \subset I$).
* $J = \{1, 2, \dots, 7\}$: Set of days in the planning horizon.
* $K = \{1, 2, 3\}$: Set of shifts per day (1 = Morning, 2 = Afternoon, 3 = Evening).

### 3.2 Parameters
* $d_{jk}$: Required number of workers on day $j$ in shift $k$.

### 3.3 Decision Variables
* $x_{ijk} \in \{0, 1\}$: 1 if worker $i$ is assigned to day $j$, shift $k$; 0 otherwise.
* $y_i \in \{0, 1\}$: 1 if worker $i$ works on both Saturday (day 6) and Sunday (day 7); 0 otherwise.
* $W_{max} \in \mathbb{Z}^+$: The maximum number of shifts worked by any single employee.
* $W_{min} \in \mathbb{Z}^+$: The minimum number of shifts worked by any single employee.

### 3.4 Objective Function
The objective utilizes Goal Programming with Penalty Costs. It minimizes the gap between the maximum and minimum shifts worked (Load Balancing), while heavily penalizing the assignment of full weekends to workers:

$$
\min \left( W_{max} - W_{min} \right) + 5 \sum_{i \in I} y_i
$$

*(Note: The weight of 5 indicates that avoiding a full weekend is prioritized over a single-shift imbalance between workers).*

### 3.5 Constraints

**Base Labor Constraints:**
* **Demand Satisfaction:** $\sum_{i \in I} x_{ijk} = d_{jk} \quad \forall j \in J, \forall k \in K$
* **Daily Workload:** Max 1 shift per day. $\sum_{k \in K} x_{ijk} \le 1 \quad \forall i \in I, \forall j \in J$
* **Weekly Workload:** Max 5 working days per week. $\sum_{j \in J} \sum_{k \in K} x_{ijk} \le 5 \quad \forall i \in I$
* **Rest-Period Constraint:** No Evening-to-Morning consecutive shifts. $x_{ij3} + x_{i(j+1)1} \le 1 \quad \forall i \in I, j \in \{1, \dots, 6\}$

**Advanced Optimization Constraints:**
* **Load Balancing Definition:** $\sum_{j \in J} \sum_{k \in K} x_{ijk} \le W_{max} \quad \forall i \in I$ and $\sum_{j \in J} \sum_{k \in K} x_{ijk} \ge W_{min} \quad \forall i \in I$
* **Skill Mix Requirement:** At least 1 Senior per shift. $\sum_{i \in S} x_{ijk} \ge 1 \quad \forall j \in J, \forall k \in K$
* **Weekend Overlap Identifier:** Forces $y_i$ to 1 if worker $i$ works at least one shift on both day 6 and day 7. $\sum_{k \in K} x_{i6k} + \sum_{k \in K} x_{i7k} - 1 \le y_i \quad \forall i \in I$

## 4. Conclusion & Output
Upon successful optimization, the script translates the MILP results into readable formats:
* `output_1.csv`: A detailed flat-file record of all assignments, tracking worker roles (Senior/Junior) and statuses.
* `output_staff.xlsx`: A matrix-style schedule (Pivot Table) providing a clear, visual roster (Shifts 1, 2, 3, or Off) for direct distribution to operations managers and staff. This demonstrates a complete pipeline from mathematical modeling to practical business reporting.
