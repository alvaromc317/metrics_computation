# `metrics_computation`

Do you usually deal with high dimensional regression datasets and you need to compute a bunch of metrics for the beta coefficients of your model?

Then `metrics_computation` is your package. We assume here that beta coefficients that are different than 0 are _positive_, while coefficients that are equal to zero are _negative_. This package can easily compute:

* `true_positive_rate`: True positive rate, sensitivity or recall = true positive / real positive
* `false_negative_rate`: False negative rate = false negative / real positive (observe thar TPR + FNR = 1)
* `true_negative_rate`: True negative rate or specificity = true negative / real negative
* `false_positive_rate`: False positive rate = false positive / real negative (TNR + FPR = 1)
* `precision`: Precision = true positive / (true positive + false positive)
* `f_score`: F-score = 2*(precision*recall)/(precision+recall)
* `correct_selection_rate`: Correct selection rate = (true positive + true negative) / total number
* `count_number_positives`: Count the number of positives in both the predicted and the real beta array
* `store_beta`: Sparse storage as an index-value pair of both predicted and real beta array

## Usage example

We first generate a synthetic dataset using the `data_generation` package available [here](https://github.com/alvaromc317/data_generation).

```
import numpy as np
import data_generation as dgen

data_equal = dgen.EqualGroupSize(n_obs=5200, ro=0.2, error_distribution='student_t', 
                                 e_df=5, random_state=1, group_size=10, non_zero_groups=7, 
                                 non_zero_coef=5, num_groups=15)

x, y, beta, group_index = data_equal.data_generation().values()
```

This generates a synthetic dataset that includes 5000 observations and 15x10=150 variables, among which 7x5=35 are different than zero and 115 are zero. Once we have the dataset, we obtain a prediction using a penalized regression model from the `asgl` package.

```
import asgl
lambda1 = 10.0 ** np.arange(-3, 1.51, 0.1)
tvt_alasso = asgl.TVT(model='lm', penalization='alasso', lambda1=lambda1, parallel=True,
                      weight_technique='pca_pct', variability_pct=0.9, error_type='MSE', 
                      random_state=1, train_size=100, validate_size=100)
alasso_result = tvt_alasso.train_validate_test(x=x, y=y)

alasso_prediction_error = alasso_result['test_error']
alasso_betas = alasso_result['optimal_betas'][1:] # Remove intercept
```

Now we have the `true_betas` and the `alasso_betas` obtained using an adaptive lasso model. Lets compute the metrics available in `metrics_computation` and see how good is our model:

```
import metrics_computation as mc
metrics_object = mc.MetricsComputation(metrics=['true_positive_rate', 'false_negative_rate', 'true_negative_rate',
                                                'false_positive_rate', 'precision', 'f_score',
                                                'correct_selection_rate', 'beta_error', 'count_number_positives',
                                                'store_beta'])
metrics = metrics_object.fit(predicted_beta=alasso_betas, true_beta=beta)
```

And the results obtained are:

```
print(metrics)
{'true_positive_rate': 1.0,
 'false_negative_rate': 0.0,
 'true_negative_rate': 0.5043478260869565,
 'false_positive_rate': 0.4956521739130435,
 'precision': 0.3804347826086957,
 'f_score': 0.5511811023622047,
 'correct_selection_rate': 0.62,
 'beta_error': 3.3391816482267616,
 'count_number_positives': {'number_positives_predicted_beta': 92,
  'number_positives_true_beta': 35},
 'store_beta': {'predicted_beta': {'index_non_zero_predicted_beta': array([  0,   1,   2,   3,   4,   5,   7,  10,  11,  12,  13,  14,  17,
           20,  21,  22,  23,  24,  25,  30,  31,  32,  33,  34,  35,  36,
           37,  38,  39,  40,  41,  42,  43,  44,  45,  46,  47,  49,  50,
           51,  52,  53,  54,  56,  57,  59,  60,  61,  62,  63,  64,  65,
           66,  67,  69,  71,  76,  77,  78,  79,  80,  85,  86,  87,  88,
           90,  91,  95,  97, 102, 103, 107, 108, 112, 114, 115, 117, 119,
          124, 125, 126, 129, 130, 133, 135, 137, 139, 140, 142, 145, 147,
          148]),
   'value_non_zero_predicted_beta': array([ 0.79339995,  1.87522975,  3.13763985,  3.61334788,  4.51621746,
           0.31125623,  0.54875971,  0.87955032,  1.81445199,  2.31443026,
           4.10932758,  5.42481166,  0.41256546,  1.12469476,  1.58801805,
           2.94509453,  3.13194602,  5.03623722,  0.13976405,  1.01355276,
           2.17043622,  2.66209677,  4.31500118,  5.35810521, -0.04337581,
          -0.10444598,  0.14715516,  0.11823591,  0.39685933,  1.16270464,
           0.66003076,  2.67933492,  3.11439734,  4.99887818,  0.4943931 ,
           0.79173587, -0.15166737,  0.01333736,  1.3140838 ,  2.29604342,
           2.42913034,  4.28719575,  4.68948029, -0.19252128,  0.09013478,
           0.0471931 ,  0.39352183,  2.59684326,  3.2423719 ,  3.70960104,
           4.47649098,  0.26081206,  0.01503026, -0.01503932,  0.18010438,
           0.3148776 , -0.19312873,  0.15733734, -0.2337122 , -0.06246009,
           0.43419795,  0.38280452,  0.00820053, -0.01194388,  0.1596238 ,
          -0.00966318,  0.00836905,  0.07707487,  0.00829384, -0.6871737 ,
           0.00859946,  0.25547944,  0.03750622,  0.20261963, -0.07859221,
          -0.05090997, -0.03631807, -0.23408105, -0.24707326, -0.10432976,
           0.01815028, -0.38785341,  0.19690068, -0.53279602, -0.1147006 ,
           0.10160788,  0.08420481, -0.34810438,  0.39998714,  0.18704431,
           0.21155179, -0.34328259])},
  'true_beta': {'index_non_zero_true_beta': array([ 0,  1,  2,  3,  4, 10, 11, 12, 13, 14, 20, 21, 22, 23, 24, 30, 31,
          32, 33, 34, 40, 41, 42, 43, 44, 50, 51, 52, 53, 54, 60, 61, 62, 63,
          64]),
   'value_non_zero_true_beta': array([1., 2., 3., 4., 5., 1., 2., 3., 4., 5., 1., 2., 3., 4., 5., 1., 2.,
          3., 4., 5., 1., 2., 3., 4., 5., 1., 2., 3., 4., 5., 1., 2., 3., 4.,
          5.])}}}
```

