.. _CoxPHmath:


Cox Proportional Hazards Model
==============================

Cox proportional hazards models are the most widely used approach for modeling
time to event data. As the name suggests, the *hazard function*, which
computes the instantaneous rate of an event occurrence and expressed
mathematically as

:math:`h(t) = \lim_{\Delta t \downarrow 0} \frac{Pr[t \le T < t + \Delta t \mid T \ge t]}{\Delta t},`

is assumed to be the product of a *baseline hazard function* and a
*risk score*. Thus the hazard function for observation :math:`i` in a Cox
proportional hazards model is defined to be

:math:`h_i(t) = \lambda(t)\exp(\mathbf{x}_i^T\beta)`

where :math:`\lambda(t)` is the baseline hazard function shared by all
observations, :math:`\exp(\mathbf{x}_i^T\beta)` is the risk score for
observation :math:`i`, which is computed as the exponentiated linear
combination of the covariate vector :math:`\mathbf{x}_i^T` using a coefficient
vector :math:`\beta` common to all observations.

This combination of a non-parametric baseline hazard function and a parametric
risk score results in Cox proportional hazards models being described as
*semi-parametric*. In addition a simple rearrangement of terms shows that
unlike in generalized linear models, an intercept (constant) term in the risk
score would add no value to the model fit due to the inclusion of a baseline
hazard function.


Defining a Cox Proportional Hazards Model
-----------------------------------------

**destination key**

  The .hex key for the resulting Cox proportional hazards model.

**source**

  The .hex key for the input data set to use in the Cox proportional hazards
  model.

**start column**

  (Optional) The name of an integer column in the **source** data set
  representing the start time. If supplied, the value of the **start column**
  must be strictly less than the **stop column** in each row.

**stop column**

  The name of an integer column in the **source** data set representing the
  stop time.

**event column**

  The name of binary data column in the **source** data set representing the
  occurrence of an event.

**x columns**

  The name of the predictor columns, both numeric and categorical, from the
  **source** data set in the Cox proportional hazards model.

**weights column**

  (Optional) The name of a numeric column in the **source** data set
  representing case weights to use in the model. If supplied, all values in the
  **weights column** must be positive.

**offset columns**

  (Optional) The name of a numeric columns in the **source** data set
  representing offset terms in the model.

**ties**

  The approximation method for handling ties in the partial likelihood; one of
  either efron or breslow. See the Cox Proportional Hazards Model Details
  section below for more information.

**init**

  (Optional) Initial values for the coefficients in the model.

**lre min**

  A positive number to use as the minimum log-relative error (LRE) of
  subsequent log partial likelihood calculations to determine algorithmic
  convergence. The role this parameter plays in the stopping criteria of the
  model fitting algorithm is explained in the Cox Proportional Hazards Model
  Algorithm section below.

**iter max**

  A positive integer denoting the maximum number of iterations to make during
  model training. The role this parameter plays in the stopping criteria of the
  model fitting algorithm is explained in the Cox Proportional Hazards Model
  Algorithm section below.


Cox Proportional Hazards Model Results
--------------------------------------

Data
~~~~
Number of Complete Cases
  The number of observations with no missing values in any of the input
  columns.
Number of Non Complete Cases
  The number of observations with at least one missing value in any of
  the input columns.
Number of Events in Complete Cases
  The number of observed events in the complete cases.

Coefficients
~~~~~~~~~~~~
:math:`\tt{name}`
  The name given to the coefficient. If the predictor column is numeric, the
  corresponding coefficient has the same name. If the predictor column is
  categorical, the corresponding coefficients are a concatenation of the name
  of the column with the name of the categorical level the coefficient
  represents.
:math:`\tt{coef}`
  The estimated coefficient value.
:math:`\tt{exp(coef)}`
  The exponentiated coefficient value estimate.
:math:`\tt{se(coef)}`
  The standard error of the coefficient estimate.
:math:`\tt{z}`
  The z statistic, which is the ratio of the coefficient estimate to its
  standard error.

Model Statistics
~~~~~~~~~~~~~~~~

Cox and Snell Generalized :math:`R^2`
  :math:`\tt{R^2} := 1 - \exp\bigg(\frac{2\big(pl(\beta^{(0)}) - pl(\hat{\beta})\big)}{n}\bigg)`

Maximum Possible Value for Cox and Snell Generalized :math:`R^2`
  :math:`\tt{Max. R^2} := 1 - \exp\big(\frac{2 pl(\beta^{(0)})}{n}\big)`

Likelihood Ratio Test
  :math:`2\big(pl(\hat{\beta}) - pl(\beta^{(0)})\big)`, which under the null
  hypothesis of :math:`\hat{beta} = \beta^{(0)}` follows a chi-square
  distribution with :math:`p` degrees of freedom.

Wald Test
  :math:`\big(\hat{\beta} - \beta^{(0)}\big)^T I\big(\hat{\beta}\big) \big(\hat{\beta} - \beta^{(0)}\big)`,
  which under the null hypothesis of :math:`\hat{beta} = \beta^{(0)}` follows a
  chi-square distribution with :math:`p` degrees of freedom. When there is a
  single coefficient in the model, the Wald test statistic value is that
  coefficient's z statistic.

Score (Log-Rank) Test
  :math:`U\big(\beta^{(0)}\big)^T \hat{I}\big(\beta^{0}\big)^{-1} U\big(\beta^{(0)}\big)`,
  which under the null hypothesis of :math:`\hat{beta} = \beta^{(0)}` follows a
  chi-square distribution with :math:`p` degrees of freedom.

where

  :math:`n` is the number of complete cases

  :math:`p` is the number of estimated coefficients

  :math:`pl(\beta)` is the log partial likelihood

  :math:`U(\beta)` is the derivative of the log partial likelihood

  :math:`H(\beta)` is the second derivative of the log partial likelihood

  :math:`I(\beta) = - H(\beta)` is the observed information matrix


Cox Proportional Hazards Model Details
--------------------------------------

A Cox proportional hazards model measures time on a scale defined by the
ranking of the :math:`M` distinct observed event occurrence times,
:math:`t_1 < t_2 < \dots < t_M`. When no two events occur at the same time,
the partial likelihood for the observations is given by

:math:`PL(\beta) = \prod_{m=1}^M\frac{\exp(w_m\mathbf{x}_m^T\beta)}{\sum_{j \in R_m} w_j \exp(\mathbf{x}_j^T\beta)}`

where :math:`R_m` is the set of all observations at risk of an event at time
:math:`t_m`. In practical terms, :math:`R_m` contains all the rows where
(if supplied) the start time is less than :math:`t_m` and the stop time is
greater than or equal to :math:`t_m`. When two or more events are observed at
the same time, the exact partial likelihood is given by

:math:`PL(\beta) = \prod_{m=1}^M\frac{\exp(\sum_{j \in D_m} w_j\mathbf{x}_j^T\beta)}{(\sum_{R^* : \mid R^* \mid = d_m} [\sum_{j \in R^*} w_j \exp(\mathbf{x}_j^T\beta)])^{\sum_{j \in D_m} w_j}}`

where :math:`R_m` is the risk set and :math:`D_m` is the set of observations
of size :math:`d_m` with an observed event at time :math:`t_m` respectively.
Due to the combinatorial nature of the denominator, this exact partial
likelihood becomes prohibitively expensive to calculate leading to the common
use of Efron's and Breslow's approximations.

**Efron's Approximation**

Of the two approximations, Efron's produces results closer to the exact
combinatoric solution than Breslow's. Under this approximation, the partial
likelihood and log partial likelihood are defined as

:math:`PL(\beta) = \prod_{m=1}^M \frac{\exp(\sum_{j \in D_m} w_j\mathbf{x}_j^T\beta)}{\big[\prod_{k=1}^{d_m}(\sum_{j \in R_m} w_j \exp(\mathbf{x}_j^T\beta) - \frac{k-1}{d_m} \sum_{j \in D_m} w_j \exp(\mathbf{x}_j^T\beta))\big]^{(\sum_{j \in D_m} w_j)/d_m}}`

:math:`pl(\beta) = \sum_{m=1}^M \big[\sum_{j \in D_m} w_j\mathbf{x}_j^T\beta - \frac{\sum_{j \in D_m} w_j}{d_m} \sum_{k=1}^{d_m} \log(\sum_{j \in R_m} w_j \exp(\mathbf{x}_j^T\beta) - \frac{k-1}{d_m} \sum_{j \in D_m} w_j \exp(\mathbf{x}_j^T\beta))\big]`

**Breslow's Approximation**

Under Breslow's approximation, the partial likelihood and log partial
likelihood are defined as

:math:`PL(\beta) = \prod_{m=1}^M \frac{\exp(\sum_{j \in D_m} w_j\mathbf{x}_j^T\beta)}{(\sum_{j \in R_m} w_j \exp(\mathbf{x}_j^T\beta))^{\sum_{j \in D_m} w_j}}`

:math:`pl(\beta) = \sum_{m=1}^M \big[\sum_{j \in D_m} w_j\mathbf{x}_j^T\beta - (\sum_{j \in D_m} w_j)\log(\sum_{j \in R_m} w_j \exp(\mathbf{x}_j^T\beta))\big]`


Cox Proportional Hazards Model Algorithm
----------------------------------------

H\ :sub:`2`\ O uses the Newton-Raphson algorithm to maximize the partial
log-likelihood, an iterative procedure defined by the steps:

0. To add numeric stability to the model fitting calculations, the numeric
   predictors and offsets are demeaned during the model fitting process.
1. Set an initial value, :math:`\beta^{(0)}`, for the coefficient vector and
   assume an initial log partial likelihood of :math:`- \infty`.
2. Increment iteration counter, :math:`n`, by 1.
3. Calculate the log partial likelihood, :math:`pl\big(\beta^{(n)}\big)`, at
   the current coefficient vector estimate.
4. Compare :math:`pl\big(\beta^{(n)}\big)` to
   :math:`pl\big(\beta^{(n-1)}\big)`.
  a) If :math:`pl\big(\beta^{(n)}\big) > pl\big(\beta^{(n-1)}\big)`, then
     accept the new coefficient vector, :math:`\beta^{(n)}`, as the current
     best estimate, :math:`\tilde{\beta}`, and set a new candidate coefficient
     vector to be :math:`\beta^{(n+1)} = \beta^{(n)} - \tt{step}`, where
     :math:`\tt{step} := H^{-1}(\beta^{(n)}) U(\beta^{(n)})`, which is the
     product of the inverse of the second derivative of :math:`pl` times the
     first derivative of :math:`pl` based upon the observed data.
  b) If :math:`pl\big(\beta^{(n)}\big) \le pl\big(\beta^{(n-1)}\big)`, then set
     :math:`\tt{step} := \tt{step} / 2` and
     :math:`\beta^{(n+1)} = \tilde{\beta} - \tt{step}`.
5. Repeat steps 2 - 4 until either
  a) :math:`n = \tt{iter\ max}` or
  b) the log-relative error
     :math:`LRE\Big(pl\big(\beta^{(n)}\big), pl\big(\beta^{(n+1)}\big)\Big) >= \tt{lre\ min}`,
     where
     :math:`LRE(x, y) = - \log_{10}\big(\frac{\mid x - y \mid}{y}\big)`, if :math:`y \ne 0`

     :math:`LRE(x, y) = - \log_{10}(\mid x \mid)`, if :math:`y = 0`


References
----------

Andersen, P. and Gill, R. (1982).
Cox's regression model for counting processes, a large sample study.
*Annals of Statistics* **10**, 1100-1120.

Harrell, Jr. F.E., Regression Modeling Strategies: With Applications
to Linear Models, Logistic Regression, and Survival Analysis.
Springer-Verlag, 2001.

Therneau, T., Grambsch, P., Modeling Survival Data: Extending the Cox Model.
Springer-Verlag, 2000.
