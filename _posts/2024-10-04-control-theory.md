---
layout: archive
title: 'Control Theory'
date: 2024-10-04
permalink: /posts/2024/10/control-theory
tags:
  - control theory
---  

## Proportional-Integral-Derivative (PID) Control
- It’s one of the simplest and most commonly used feedback control algorithms.

- The controller continuously calculates an error value as the difference between a desired setpoint and a measured process variable, and applies corrections based on proportional, integral, and derivative terms.

  - Proportional (P): Corrects the error proportionally to its size (fast response but may not eliminate error).
  - Integral (I): Accounts for past errors to eliminate steady-state error.
  - Derivative (D): Predicts future error by assessing the rate of change (used to smooth the response).

- Applications:
PID controllers are used in many simple systems where precise tuning of these three parameters can yield good control performance. For example, temperature control, speed control, etc.

- Pros:
Simple to implement.
Suitable for single-input single-output (SISO) systems.
Widely supported in control systems and easy to tune for many practical processes.

- Cons:
Limited in handling multivariable (MIMO) systems.
No consideration of constraints (e.g., actuator limits).
Can be difficult to tune for systems with delays or high-order dynamics.


## Model Predictive Control (MPC)
Overview:
MPC is an advanced control strategy that uses a dynamic model of the system to predict future states over a specified horizon and optimizes control actions by solving a constrained optimization problem.
It is able to handle multi-variable systems with constraints on inputs and outputs.
Working:
Prediction Horizon: MPC predicts future behavior using a model of the system over a finite time horizon.
Optimization: It solves an optimization problem at every step to minimize a cost function (often a combination of tracking error and control effort), considering system constraints.
Receding Horizon: Only the first control action is applied, and the process is repeated in the next time step.
Applications:
MPC is widely used in chemical processes, robotics, autonomous driving, and energy systems, where there are complex dynamics, multiple inputs/outputs, and operational constraints.
Pros:
Can handle multi-variable (MIMO) systems.
Optimizes performance subject to constraints (e.g., physical limits, safety constraints).
Adapts well to non-linear and complex systems.
Cons:
Computationally intensive (requires solving optimization problems in real-time).
More complex to implement and requires a dynamic model of the system.
Less intuitive to tune than PID.


## State-Space Control
State-Space Control is based on modeling the system in a state-space form, where the system dynamics are described using state variables and a set of linear differential equations.
Typically used for systems where multiple states are involved, and it is suitable for multi-input multi-output (MIMO) systems.

### Working

**State-Space Representation:** The system is represented as a set of first-order differential (or difference) equations:


x˙(t) = A x(t) + B u(t)
y(t) = C x(t) + D u(t)


where `x(t)` represents the system states, `u(t)` is the input, `y(t)` is the output, and `A`, `B`, `C`, and `D` are matrices defining system dynamics.

**Control Design:** A state-feedback controller computes the input based on the full state vector, usually with a Linear Quadratic Regulator (LQR) or Pole Placement approach.

### Applications

State-Space Control is common in aerospace, electrical systems, and mechanical systems, where complex dynamics and multiple states need to be controlled.

### Pros
* Well-suited for multi-variable (MIMO) systems.
* Can handle internal dynamics (states that aren’t directly observable).
* Provides a rigorous framework for analyzing stability and controllability.

### Cons
* Requires an accurate model of the system.
* State measurements or estimations (via observers like a Kalman filter) may be required.
* Can be mathematically intensive and harder to implement than PID.


## Example: Lunar Lander 

1. PID Approach Explanation:
The vertical PID controller tries to maintain a safe vertical speed by adjusting thrust based on the error between the current vertical speed and the target (-40 m/s).
The horizontal PID controller tries to stabilize horizontal speed by adjusting the rotation of the lander.

```python
class PIDController:
    def __init__(self, kp: float, ki: float, kd: float):
        self.kp = kp
        self.ki = ki
        self.kd = kd
        self.prev_error = 0
        self.integral = 0

    def compute(self, target: float, current: float, dt: float):
        error = target - current
        self.integral += error * dt
        derivative = (error - self.prev_error) / dt
        self.prev_error = error
        return self.kp * error + self.ki * self.integral + self.kd * derivative


# PID parameters (tuned for vertical speed and horizontal speed)
vertical_pid = PIDController(kp=0.2, ki=0.0, kd=0.5)
horizontal_pid = PIDController(kp=0.1, ki=0.0, kd=0.3)

# Main loop
while True:
    x, y, h_speed, v_speed, fuel, rotate, power = [int(i) for i in input().split()]

    # Vertical control: target vertical speed is a soft landing (40 m/s)
    target_v_speed = -40  # We want to limit vertical speed to -40 m/s
    dt = 1  # Time step (1 second)
    vertical_thrust = vertical_pid.compute(target_v_speed, v_speed, dt)
    vertical_thrust = min(max(vertical_thrust, 0), 4)  # Clamp thrust power between 0 and 4

    # Horizontal control: try to reduce horizontal speed to 0
    target_h_speed = 0
    horizontal_rotation = horizontal_pid.compute(target_h_speed, h_speed, dt)
    horizontal_rotation = min(max(horizontal_rotation, -90), 90)  # Clamp rotation between -90 and 90

    # Output the computed rotation and thrust
    print(f"{int(horizontal_rotation)} {int(vertical_thrust)}")
```


2. Model Predictive Control (MPC)
MPC is a more advanced approach that involves predicting future system behavior based on the current state and optimizing over a prediction horizon. In this case, we will:

Predict future positions, velocities, and adjust the rotation and thrust accordingly.
Solve a constrained optimization problem to minimize the error in vertical and horizontal speed while landing.
This is more complex to implement as it requires solving an optimization problem at each step.

```python
import numpy as np
from scipy.optimize import minimize

# Define system dynamics for Mars lander
def dynamics(state, control):
    x, y, h_speed, v_speed, fuel = state
    rotate, thrust = control

    # Simulate the next state based on dynamics (gravity + thrust)
    g = 3.711
    thrust_force = thrust * np.cos(np.radians(rotate))
    h_accel = -thrust * np.sin(np.radians(rotate))
    v_accel = thrust_force - g
    
    # Next state
    next_h_speed = h_speed + h_accel
    next_v_speed = v_speed + v_accel
    next_x = x + h_speed + 0.5 * h_accel
    next_y = y + v_speed + 0.5 * v_accel
    next_fuel = fuel - thrust
    return [next_x, next_y, next_h_speed, next_v_speed, next_fuel]

# Define cost function to minimize
def cost_function(control_sequence, initial_state, target_v_speed=-40, target_h_speed=0):
    state = initial_state
    cost = 0
    for control in control_sequence:
        state = dynamics(state, control)
        h_speed, v_speed = state[2], state[3]
        cost += (h_speed - target_h_speed) ** 2 + (v_speed - target_v_speed) ** 2  # Penalize deviation from targets
    return cost

# Solve MPC at each step
def solve_mpc(initial_state):
    horizon = 10
    control_guess = [(0, 0)] * horizon  # Initial guess (rotate, thrust)
    bounds = [(-90, 90), (0, 4)] * horizon  # Control limits

    res = minimize(cost_function, control_guess, args=(initial_state,), bounds=bounds)
    return res.x[0]  # Return first control (rotate, thrust)

# Main loop
while True:
    x, y, h_speed, v_speed, fuel, rotate, power = [int(i) for i in input().split()]
    state = [x, y, h_speed, v_speed, fuel]
    
    # Solve the MPC problem
    rotate, thrust = solve_mpc(state)
    
    # Output the first control action
    print(f"{int(rotate)} {int(thrust)}")

```
MPC Approach Explanation:
We define a cost function that penalizes deviation from desired horizontal and vertical speeds.
System dynamics are simulated to predict the next states based on the current control input.
We use a numerical optimization method (like scipy.optimize.minimize) to minimize this cost over a prediction horizon, giving us the optimal sequence of controls.
At each step, we apply the first control action and repeat.

3. State-Space Control

For State-Space Control, we need to represent the lander's dynamics in state-space form and design a state-feedback controller, like an LQR (Linear Quadratic Regulator).

**State-Space Formulation:**

State vector: $[x, y, h\_speed, v\_speed, fuel]$

Control vector: $[rotate, thrust]$

```python
import numpy as np
from scipy.linalg import solve_continuous_are

# Define system dynamics matrices (A, B, C, D for state-space)
A = np.array([
    [1, 0, 1, 0, 0],
    [0, 1, 0, 1, 0],
    [0, 0, 1, 0, 0],
    [0, 0, 0, 1, 0],
    [0, 0, 0, 0, 1]
])

B = np.array([
    [0, 0],
    [0, 0],
    [1, 0],
    [0, 1],
    [0, 0]
])

# Define Q, R matrices for LQR
Q = np.diag([1, 1, 10, 10, 0])  # Penalize speed errors more
R = np.diag([1, 1])  # Penalize large control inputs

# Solve the continuous-time algebraic Riccati equation for the optimal feedback gain
P = solve_continuous_are(A, B, Q, R)
K = np.linalg.inv(R) @ B.T @ P  # Feedback gain

# Main loop
while True:
    x, y, h_speed, v_speed, fuel, rotate, power = [int(i) for i in input().split()]
    state = np.array([x, y, h_speed, v_speed, fuel])

    # Compute the control action using state feedback
    control = -K @ state
    rotate = np.clip(control[0], -90, 90)
    thrust = np.clip(control[1], 0, 4)

    # Output the control action
    print(f"{int(rotate)} {int(thrust)}")

```
**State-Space Approach Explanation:**

The system is modeled in a state-space form using matrices $A$, $B$, etc.

An LQR controller is designed to compute optimal control actions based on the current state, minimizing a cost that penalizes large deviations from desired speeds and large control inputs.

The feedback gain $K$ is computed, and controls are applied based on the current state.


Dynamic programming (DP) and planning are related to control strategies like PID, MPC, and State-Space control in the sense that they provide a way to optimize decision-making over time. Specifically, they deal with breaking down complex problems into simpler sub-problems and then solving them in an optimal manner. In the context of the Mars Lander problem, dynamic programming and planning approaches could be used to find optimal sequences of actions (thrust and rotation) that minimize some cost (e.g., fuel usage or speed deviations) while ensuring a safe landing.

4. Dynamic Programming (DP)
Dynamic programming is typically used for solving optimization problems by breaking them down into sub-problems. It is particularly effective when:

The problem can be decomposed into stages.
The problem has overlapping sub-problems.
The problem exhibits the principle of optimality: an optimal solution to the problem can be constructed from optimal solutions to its sub-problems.

Applying DP to Mars Lander:
The goal is to minimize fuel consumption while ensuring that the lander touches down safely (with low vertical and horizontal speed). DP can be used to find the optimal thrust and rotation sequence over time, taking into account:

The current state (position, speed, fuel, etc.).
Future states based on the decisions made (thrust and rotation).
A state in the Mars Lander problem can be represented as:
\text{State} = (x, y, h_{\text{speed}}, v_{\text{speed}}, \text{fuel})

The key is to compute the optimal action (rotation and thrust) from any state to reach the goal state (successful landing) with minimum cost.

DP Approach to Solve Mars Lander:
Define State Space:

Each state is characterized by the lander's position, speed, and fuel.
Each action (control) changes the state according to the system's dynamics.
Define Transition Function:

For each state, simulate how the lander will transition to the next state based on current thrust and rotation.
Define Cost Function:

The cost can represent various objectives:
Fuel consumption (minimize fuel used).
Time taken (minimize the time to land).
Penalties for exceeding speed limits (horizontal and vertical).
Recursively Calculate Optimal Solutions:

Use backward recursion to compute the optimal policy (sequence of actions) that minimizes the cost from each state.
Apply Memoization/Tabulation:

Store the results of sub-problems (state-action pairs) to avoid recalculating them, thus speeding up the process.

```python
import math

# Constants
GRAVITY = 3.711

# Define the state as (x, y, h_speed, v_speed, fuel)
# This would typically be a large state space, so discretization might be needed.

# Define possible actions: thrust (0 to 4), rotate (-90 to 90 degrees)
possible_thrusts = [0, 1, 2, 3, 4]
possible_rotations = range(-90, 91, 15)

# Define a function for simulating the next state based on current state and action
def simulate_next_state(state, action):
    x, y, h_speed, v_speed, fuel = state
    rotation, thrust = action

    # Horizontal and vertical accelerations
    h_accel = -thrust * math.sin(math.radians(rotation))
    v_accel = thrust * math.cos(math.radians(rotation)) - GRAVITY

    # Next state
    next_h_speed = h_speed + h_accel
    next_v_speed = v_speed + v_accel
    next_x = x + h_speed + 0.5 * h_accel
    next_y = y + v_speed + 0.5 * v_accel
    next_fuel = max(fuel - thrust, 0)

    return (next_x, next_y, next_h_speed, next_v_speed, next_fuel)

# Define a cost function (simplified fuel consumption)
def cost(state, action):
    _, _, _, _, fuel = state
    rotation, thrust = action
    return thrust  # Simple cost: more thrust = higher cost (fuel consumption)

# Dynamic programming to find the optimal action from each state
def dp_optimal_landing(initial_state, goal_state, time_horizon):
    # Memoization table to store the cost-to-go and optimal action for each state
    dp_table = {}

    def dp(state, t):
        if state in dp_table:
            return dp_table[state]

        # Base case: if we are at the goal state (safe landing)
        if t == time_horizon or (state[1] <= goal_state[1] and abs(state[2]) <= 20 and abs(state[3]) <= 40):
            return 0, None  # No cost and no further action

        # Recursive case: minimize cost over all possible actions
        best_cost = float('inf')
        best_action = None
        for thrust in possible_thrusts:
            for rotation in possible_rotations:
                action = (rotation, thrust)
                next_state = simulate_next_state(state, action)
                future_cost, _ = dp(next_state, t + 1)
                current_cost = cost(state, action) + future_cost
                if current_cost < best_cost:
                    best_cost = current_cost
                    best_action = action

        dp_table[state] = (best_cost, best_action)
        return dp_table[state]

    # Start from the initial state
    return dp(initial_state, 0)

# Example usage
initial_state = (2500, 2700, 0, 0, 500)  # Starting state: x, y, h_speed, v_speed, fuel
goal_state = (2500, 0, 0, 0, 0)  # Goal state: safe landing
optimal_cost, optimal_action = dp_optimal_landing(initial_state, goal_state, 100)
print(f"Optimal cost: {optimal_cost}, First action: {optimal_action}")
```

How DP Relates to the Other Methods:
Similar to MPC, dynamic programming solves an optimization problem by considering future states. However, instead of solving a numerical optimization problem at each step, DP precomputes an optimal sequence for all states and actions in the state space.
More exhaustive than PID, since DP considers the long-term future by optimizing the entire landing trajectory, whereas PID operates based on current errors.
In contrast to state-space methods, DP explores all possible state transitions instead of relying on linear feedback control.


5. Planning
Planning involves reasoning over sequences of actions to achieve a goal. In the Mars Lander context, planning could mean searching for a sequence of thrusts and rotations that ensure a safe landing. Popular planning algorithms are A*, Dijkstra's algorithm, or Policy Iteration in reinforcement learning. These are often used to solve problems where the state space is large but discrete.

How Planning Differs from DP:
DP computes an optimal solution for every state, while planning algorithms like A* search for an optimal path from the initial state to the goal state.
Planning can be more focused: rather than solving for all possible states, a planning algorithm searches for a specific sequence of actions to reach the goal, often using heuristics to speed up the search.
A* Search for Mars Lander:
In A*, we would define a heuristic to estimate the cost to reach the goal (e.g., vertical distance left to travel, or remaining fuel) and search for a sequence of actions that minimizes fuel consumption while safely landing.

```python
import heapq

def heuristic(state, goal_state):
    # Heuristic: Euclidean distance to goal + penalize speed deviations
    x, y, h_speed, v_speed, fuel = state
    return abs(y - goal_state[1]) + abs(h_speed) + abs(v_speed)

def a_star_search(initial_state, goal_state):
    open_set = []
    heapq.heappush(open_set, (0, initial_state, []))  # (cost, state, path)

    visited = set()

    while open_set:
        current_cost, current_state, current_path = heapq.heappop(open_set)

        if current_state in visited:
            continue
        visited.add(current_state)

        # Check if we reached the goal
        if current_state[1] <= goal_state[1] and abs(current_state[2]) <= 20 and abs(current_state[3]) <= 40:
            return current_path

        # Explore next states
        for thrust in possible_thrusts:
            for rotation in possible_rotations:
                action = (rotation, thrust)
                next_state = simulate_next_state(current_state, action)
                new_cost = current_cost + cost(current_state, action) + heuristic(next_state, goal_state)
                heapq.heappush(open_set, (new_cost, next_state, current_path + [action]))

# Example usage
initial_state = (2500, 2700, 0, 0, 500)
goal_state = (2500, 0, 0, 0, 0)
optimal_path = a_star_search(initial_state, goal_state)
print(f"Optimal path

```


## References
- https://www.youtube.com/playlist?list=PLxdnSsBqCrrF9KOQRB9ByfB0EUMwnLO9o