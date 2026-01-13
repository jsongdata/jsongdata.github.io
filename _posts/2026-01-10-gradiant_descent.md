---
title: "Understanding Gradient Descent in Deep Learning"
date: 2025-01-12
categories: [DeepLearning]
tags: [deep learning]
---

# Understanding Derivatives and Gradient Descent in Deep Learning

### 1. Mathematical Definition
* **Derivative Coefficient (미분계수)**: This refers to the numerical value representing the **rate of change** of a function's value at a specific point, or the **slope of the tangent line** at that point.
* In Deep Learning, calculating this value is an absolute requirement for executing **Gradient Descent**.

### 2. Derivative = 0 vs. Gradient Descent
Why do we use Gradient Descent instead of simply finding the point where the derivative is zero?

* **Mathematical Ideal (Closed-form Solution)**: For simple functions like a quadratic equation, we can solve $f'(x) = 0$ algebraically to find the minimum in one step.
* **The Reality of Deep Learning**:
    * **Not a Closed-form**: Most loss functions in DL cannot be solved with a simple formula.
    * **Complexity**: Loss functions involve millions of parameters ($W$, $b$), making it computationally impossible to solve for zero all at once.
    * **Efficiency**: When dealing with massive amounts of data, Gradient Descent is far more computationally efficient.

### 3. The Logic of Gradient Descent
The direction of the update depends on the sign of the derivative:

* **If the derivative is positive (+)**:
    * The function value is increasing.
    * **Action**: Move in the **opposite direction** (decrease $x$) to find the minimum.
* **If the derivative is negative (-)**:
    * The function value is decreasing.
    * **Action**: **Continue in that direction** (increase $x$) to reach the minimum.

### 4. The Essence of Training
Ultimately, training a deep learning model is the continuous process of updating numerous **weights ($W$)** by calculating the derivatives of the loss function until the gradient reaches near zero (the minimum point).