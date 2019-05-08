# Udacity-A-B-Testing-Final-Project

---

- [1. Experiment Overview: Free Trial Screener](#1-experiment-overview)
- [2. Choosing Metrics](#2-choosing-metrics)
  - [2.1 Choosing Invariant Metrics](#21-invariant-metrics)
  - [2.2 Choosing Variant Metrics](#22-variant-metrics)
  - [2.3 Measuring Variability](#23-measuring-variability)
- [3. Designing Experiment]()
  - [3.1 Sizing - Number of Pageviews]()
  - [3.2 Duration and Exposure]()
- [4. Analyzing Results]()
  - [4.1 Sanity Checks]()
  - [4.2 Effect Size Tests]()
  - [4.3 Sign Test]()
  
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

* **Gross conversion:** number of user-ids to complete checkout and enroll in the free trial divided by number of unique cookies to click the "Start free trial" button.
* **Retention:** number of user-ids to remain enrolled past the 14-day boundary (andthus make at least one payment) divided by number of user-ids to complete checkout.
* **Net conversion:** number of user-ids to remain enrolled past the 14-dayboundary (and thus make at least one payment) divided by the number of unique cookies to click the "Start free trial" button.

### 2.3 Measuing Variability  

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

The standard variability of gross conversion is 0.0202, retention is 0.0549, and net conversion is 0.0156.
