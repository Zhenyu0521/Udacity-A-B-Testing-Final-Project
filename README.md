# Udacity-A-B-Testing-Final-Project

---

- [1. Experiment Overview: Free Trial Screener](#1-experiment-overview)
- [2. Choosing Metrics](#2-choosing-metrics)
  - [2.1 Choosing Invariant Metrics](#21-invariant-metrics)
  - [2.2 Choosing Variant Metrics](#22-variant-metrics)
  - [2.3 Measuring Variability](#23-measuring-variability)
- [3. Designing Experiment](#3-designing-experiment)
  - [3.1 Sizing - Number of Cookies](#31-sizing-number-of-cookies)
  - [3.2 Duration and Exposure](#32-duration-and-exposure)
- [4. Analyzing Results](#4-analyzing-results)
  - [4.1 Sanity Checks](#41-sanity-checks)
    - [4.1.1 Pageviews](#411-pageviews)
    - [4.1.2 Clicks](#412-clicks)
    - [4.1.3 Click-through-probability](#413-click-through-probability)
  - [4.2 Check for Statistical and Practical Significance](#42-check-for-statistical-and-practical-significance)
    - [4.2.1 Gross conversion](#421-gross-conversion)
    - [4.2.2 Net conversion](#422-net-conversion)
  - [4.3 Sign Test](#43-sign-test)
    - [4.3.1 Gross conversion](#431-gross-conversion)
    - [4.3.2 Net conversion](#432-net-conversion)
- [5. Conclusion](#5-conclusion)
  
  
---

## 1. Experiment Overview
This project is the final project of Udacity A/B Testing. It includes the whole process of designing an A/B Testing, from selecting metrics to analyzing results finally.

At the time of this experiment, Udacity courses currently have two options on the course overview page: "start free trial", and "access course materials". If the student clicks "start free trial", they will be asked to enter their credit card information, and then they will be enrolled in a free trial for the paid version of the course. After 14 days, they will automatically be charged unless they cancel first. If the student clicks "access course materials", they will be able to view the videos and take the quizzes for free, but they will not receive coaching support or a verified certificate, and they will not submit their final project for feedback.

In the experiment, Udacity tested a change where if the student clicked "start free trial", they were asked how much time they had available to devote to the course. If the student indicated 5 or more hours per week, they would be taken through the checkout process as usual. If they indicated fewer than 5 hours per week, a message would appear indicating that Udacity courses usually require a greater time commitment for successful completion, and suggesting that the student might like to access the course materials for free. At this point, the student would have the option to continue enrolling in the free trial, or access the course materials for free instead. The following screenshot shows what the experiment looks like.

![screenshot](https://github.com/Zhenyu0521/Udacity-A-B-Testing-Final-Project/blob/master/Data/Final%20Project_%20Experiment%20Screenshot.png)

The hypothesis was that this might set clearer expectations for students upfront, thus reducing the number of frustrated students who left the free trial because they didn't have enough time—without significantly reducing the number of students to continue past the free trial and eventually complete the course. If this hypothesis held true, Udacity could improve the overall student experience and improve coaches' capacity to support students who are likely to complete the course.


The unit of diversion is a cookie, although if the student enrolls in the free trial, they are tracked by user-id from that point forward. The same user-id cannot enroll in the free trial twice. For users that do not enroll, their user-id is not tracked in the experiment, even if they were signed in when they visited the course overview page.

## 2. Choosing Metrics

### 2.1 Invariant Metrics

* **Number of cookies:** number of unique cookies to view the course overview page.
* **Number of clicks:** number of unique cookies to click the "Start free trial" button (which happens before the free trial screener is trigger).
* **Click-through-probability:** number of unique cookies to click the "Start free trial" button divided by number of unique cookies to view the course overview page.

### 2.2 Variant Metrics

* **Gross conversion:** number of user-ids to complete checkout and enroll in the free trial divided by number of unique cookies to click the "Start free trial" button. (dmin = 0.01)
* **Retention:** number of user-ids to remain enrolled past the 14-day boundary (and thus make at least one payment) divided by number of user-ids to complete checkout. (dmin = 0.01)
* **Net conversion:** number of user-ids to remain enrolled past the 14-dayboundary (and thus make at least one payment) divided by the number of unique cookies to click the "Start free trial" button. (dmin = 0.0075)

### 2.3 Measuing Variability  

We need to make an analytic estimate of evaluation metric's standard deviation based on historical data. In the further step, we need these standard deviations to calculate sample size.

```python
# Given a sample size of 5000 cookies
baseline = {'Cookies': 5000,
            'Click': 400,
            'Enrollment': 82.5,
            'CTP': 0.08,
            'GC': 0.20625,
            'R': 0.53,
            'NC': 0.1093125}

std_GC = np.sqrt(baseline['GC']*(1-baseline['GC'])/baseline['Click'])
std_R  = np.sqrt(baseline['R']*(1-baseline['R'])/baseline['Enrollment'])
std_NC = np.sqrt(baseline['NC']*(1-baseline['NC'])/baseline['Click'])
print("The standard variability of gross conversion is {}, retention is {}, and net conversion is {}".format(std_GC, std_R, std_NC))
```

The standard deviation of gross conversion is 0.0202, retention is 0.0549, and net conversion is 0.0156.

## 3. Designing Experiment

During the process of designing experiment, we need to determine how many samples we need to collect. And we also should consider the duration of the experiment.

### 3.1 Sizing - Number of Cookies

Sample size is generally related to significance level, type II error, baseline conversion, practical significance and so. And the [Online_Sample_Size_Calculator](http://www.evanmiller.org/ab-testing/sample-size.html) is a convenient method to calculate the sample size. alpha = 0.5, beta = 0.2

```python
index   = ['GC', 'R', 'NC']
columns = ['Sample Size', 'Number of Cookies']
data    = [[25835, round(2*25835/baseline['CTP'])], 
           [39115, round(2*39115/baseline['GC']/baseline['CTP'])],
           [27413, round(2*27413/baseline['CTP'])]]
sample_size = pd.DataFrame(index=index, columns=columns, data=data)
sample_size
```

|   |Sample_size|Number_of_cookies|
|---|---|---|
|GC|25835|645875|
|R|39115|4741212|
|NC|27413|685325|

### 3.2 Duration and Exposure

For retension, we need at least 4741212 cookies, which means we need to collect 119 days' data given the condition that we only get 40000 cookies per day. The duration is too long and the result cannot be accurate enough. Thus we drop this evaluation metric in this experiment. By doing this, we then need only 685325 cookies to perform our test.

Also, we do not want to expose all our traffic to this experiment, because the experiment might cause side-effect on our users. Thus, we choose to divert 50% of our traffic to this experiment. Thus, it will take us roughly a month for this experiment. This is a more realistic duration.

## 4. Analyzing Results

### 4.1 Sanity Checks

Before our final tests, we need to perform a sanity check to make our invariant metrics are equivalent between the two groups, since the changes we make will not affect those metrics.

#### 4.1.1 Pageviews

```python
pageviews_control = control.Pageviews.sum()
pageviews_treatment = treatment.Pageviews.sum()
pageviews_total = pageviews_control + pageviews_treatment
stand_pageviews = (pageviews_control/pageviews_total - 0.5)/np.sqrt((0.5**2/pageviews_total))
p_pageviews = 1-stats.norm.cdf(stand_pageviews) 
p_pageviews
```
p_value is 0.1439, which is larger than 0.05. Therefore, we cannot reject the null hypothesis that pageviews are the same between control and treatment group.

Another way to do sanity check is the following:

```python
sd_pageviews = np.sqrt(0.5*(1-0.5)/(pageviews_control+pageviews_treatment))
me_pageviews = sd_pageviews*1.96
ci_pageviews = [0.5-me_pageviews, 0.5+me_pageviews]
prop_pageviews = pageviews_control/pageviews_total
print(ci_pageviews, prop_pageviews)
```

The confidence interval of pageviews is (0.4988, 0.5012), and proportion of pageviews in control group is 0.5006, which is in the range of the confidence interval. Therefore, pageviews are the same between control and treatment group.

#### 4.1.2 Clicks

```python
click_control = control.Clicks.sum()
click_treatment = treatment.Clicks.sum()
click_total = click_control + click_treatment
stand_click = (click_control/click_total - 0.5)/np.sqrt((0.5**2/click_total))
p_click = 1-stats.norm.cdf(stand_click) 
p_click
```

p_value is 0.4119, which is larger than 0.05. Therefore, we cannot reject the null hypothesis that clicks are the same between control and treatment group.

#### 4.1.3 Click-through-probability

```python
ctp_control = click_control/pageviews_control
ctp_treatment = click_treatment/pageviews_treatment
ctp_pool = click_total/pageviews_total
ctp_std = np.sqrt(ctp_pool*(1-ctp_pool)*(1/pageviews_control + 1/pageviews_treatment))
stand_ctp = (ctp_control-ctp_treatment)/ctp_std
p_ctp = stats.norm.cdf(stand_ctp) 
p_ctp
```

p_value is 0.4659, which is larger than 0.05. Therefore, we cannot reject the null hypothesis that CTR are the same between control and treatment group.

Thus, our experiment passes the sanity check. We can move forward and start testing the effect of the changes we make.

### 4.2 Check for Statistical and Practical Significance

#### 4.2.1 Gross conversion

```python
# Numbers of enrollments in two groups
enroll_control   = control.Enrollments.sum()
enroll_treatment = treatment.Enrollments.sum()
# Nubmers of clicks in two groups
click_control = control.loc[control.Enrollments.notnull(), 'Clicks'].sum()
click_treatment= treatment.loc[treatment.Enrollments.notnull(), 'Clicks'].sum()
# Gross conversion of two groups
gc_control   = enroll_control/click_control
gc_treatment = enroll_treatment/click_treatment
# p_value
gc_diff   = gc_treatment - gc_control
gc_pooled = (enroll_control + enroll_treatment)/(click_control + click_treatment)
gc_sd     = np.sqrt(gc_pooled*(1-gc_pooled)*(1/click_control + 1/click_treatment))
stand_gc  = gc_diff/gc_sd
p_gc      = stats.norm.cdf(stand_gc)
p_gc
```

```python
ci_gc = [gc_diff-stats.norm.ppf(1-0.05/2)*gc_sd, gc_diff+stats.norm.ppf(1-0.05/2)*gc_sd]
ci_gc
```

The p-value of difference between the control and treatment groups is 1.2892005168602965e-06, which is smaller than 0.05. Therefor the difference is statistically significant. And the minimum detectable difference of gross conversion (0.01) is in the range of confidence intervals (-0.0291, -0.0120), which means the difference is also practically significant. Thus, we can say the screener does effectively reduce the number of students who enrolled at the initial click.

#### 4.2.2 Net conversion

```python
# Numbers of payments in two groups
pay_control   = control.Payments.sum()
pay_treatment = treatment.Payments.sum()
# Nubmers of clicks in two groups
click_control = control.loc[control.Payments.notnull(), 'Clicks'].sum()
click_treatment= treatment.loc[treatment.Payments.notnull(), 'Clicks'].sum()
# Net conversion of two groups
nc_control   = pay_control/click_control
nc_treatment = pay_treatment/click_treatment
# p_value
nc_diff   = nc_treatment - nc_control
nc_pooled = (pay_control + pay_treatment)/(click_control + click_treatment)
nc_sd     = np.sqrt(nc_pooled*(1-nc_pooled)*(1/click_control + 1/click_treatment))
stand_nc  = nc_diff/nc_sd
p_nc      = stats.norm.cdf(stand_nc)
p_nc
```

```python
ci_nc = [nc_diff-stats.norm.ppf(1-0.05/2)*nc_sd, nc_diff+stats.norm.ppf(1-0.05/2)*nc_sd]
ci_nc
```

The p-value of net conversion is 0.0779, which is larger than 0.05. Therefore, we cannot reject the null hypothesis and the difference between two groups is not statistically significant. The confidence interval is (-0.0116, 0.0019) and it does not contain the minimum detectable difference (0.0075), which means it's not practically significant. Thus, for net conversion, the difference is not statistically and practically significant.

### 4.3 Sign Test

```python
c_t = control.merge(treatment, left_index=True, right_index=True)
c_t = c_t.loc[c_t.Payments_x.notnull(),:]
c_t['GC'] = c_t['Enrollments_y']/c_t['Clicks_y'] > c_t['Enrollments_x']/c_t['Clicks_x']
c_t['NC'] = c_t['Payments_y']/c_t['Clicks_y'] > c_t['Payments_x']/c_t['Clicks_x']

def sign_test(x, n):
    def binomial(x, n):
        return comb(n,x)*0.5**n
    p = 0
    for i in range(x+1):
        p = p + binomial(i, n)
    return p
```

#### 4.3.1 Gross conversion

```python
p_gc_sign = 2 * sign_test(c_t['GC'].sum(), c_t['GC'].count())
p_gc_sign
```

#### 4.3.2 Net conversion

```python
p_nc_sign = 2 * sign_test(c_t['NC'].sum(), c_t['NC'].count())
p_nc_sign
```

The results of sign tests are consistent with statistical and practical significance tests. The change in gross conversion is indeed siginificant while the change in net conversion is not.

## 5. Conclusion

Based on the result, I would not launch this change. Because the change in net conversion is not significant, which means though gross conversion is reduces while it does not change net conversion. Also, further experiments need to be performed to test other metrics.


