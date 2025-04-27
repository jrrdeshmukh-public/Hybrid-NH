
Implementation Notes
====================

Because the objective function now includes an absolute value, standard linear programming solvers are not directly applicable. The following approaches can be used for implementation:

1\. Brute-Force Search (for small problem instances)
----------------------------------------------------

For very small problem instances (small number of time steps _T_ and limited range of values for the decision variables), a brute-force search can be used. This involves:

*   Discretizing the decision variables ($w\_e(t)$, $w\_{hp}(t)$, $w\_{ch}(t)$, $w\_{er}(t)$ ) into a finite set of possible values.
*   Evaluating the objective function for all possible combinations of the discretized variables.
*   Selecting the combination that yields the minimum offset.

This approach is computationally expensive and quickly becomes infeasible as the problem size increases.

2\. Linearization with Auxiliary Variables
------------------------------------------

The absolute value can be linearized by introducing auxiliary variables. For each time step _t_, introduce two non-negative variables, $s(t)$ (surplus) and $sh(t)$ (shortage), and the following constraints:

    s(t) - sh(t) = Celectricity(t) + wch(t) * Chydrogen(t) - (we(t) * En * ηn + wer(t) * Hr(t-1) * Yr)
    s(t) >= 0
    sh(t) >= 0
    

The objective function then becomes:

    Minimize Offset = ∑t=1T (s(t) + sh(t) - SV(t) * Hr(t))
    

This formulation is now entirely linear and can be solved using standard linear programming solvers.

3\. Non-Linear Programming Solvers (with Computed Gradients)
------------------------------------------------------------

For larger problem instances, using non-linear programming (NLP) solvers is a more efficient approach. These solvers can directly handle the absolute value in the objective function. Crucially, providing the solver with the _gradients_ of the objective function can greatly improve performance. The gradients represent the rate of change of the objective function with respect to each decision variable.

Calculating the gradients for the objective function with the absolute value requires careful consideration of the cases where the expression inside the absolute value is positive or negative. The gradient for a particular time step _t_ will be:

    ∂Offset/∂x(t) = 
      ∂/∂x(t) (Consumption(t) - Production(t) - SV(t) * Hr(t))   if Consumption(t) >= Production(t)
      ∂/∂x(t) (Production(t) - Consumption(t) - SV(t) * Hr(t))   if Consumption(t) < Production(t)
    

Where $x(t)$ represents any of the decision variables at time $t$.

Using an NLP solver with computed gradients is generally the most efficient method for solving this type of problem for realistic problem sizes. Examples of suitable solvers include IPOPT, SNOPT, and KNITRO.

Hyperparameter Values
---------------------

The following are example hyperparameter values that can be used for testing and experimentation. These values are illustrative and may need to be adjusted based on the specific application and data.

*   $E\_n$ (Total nuclear energy capacity): 3000 MWh
*   $η\_n$ (Efficiency of nuclear electricity generation): 0.35
*   $Y\_h$ (Conversion efficiency of nuclear energy to hydrogen): 0.02 kg/MWh
*   $Y\_r$ (Conversion efficiency of hydrogen reconversion to electricity): 0.033 MWh/kg
*   $C\_0$ (Baseline electricity demand): 2000 MWh
*   $A$ (Amplitude of daily electricity demand fluctuation): 0.2
*   $C\_0$ (Baseline hydrogen demand): 5000 kg
*   $A\_h$ (Amplitude of seasonal hydrogen demand fluctuations): 0.1
*   $g$ (Growth rate of hydrogen's value): 0.0001 per 30-minute interval
*   $r$ (Discount rate applied to hydrogen’s future value): 0.00005 per 30-minute interval
*   $T$ (Total time steps): 48 (representing 24 hours in 30-minute intervals)
