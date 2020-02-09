---
layout: post
title:  "How to develop a machine learning solution with TDD"
date:   2019-10-02 
categories: 
---

Machine learning algorithms are based on stochastic process, which makes it freakingly difficult to test.

From the Kent Becks 'Test Driven Development by Example', to test the implementation itself is not recommended. 
We need to test the behavior, or the interface of the program. This is called Behavior Driveen Development to 
divide its word from the TDD.

As you can check on this [link](https://towardsdatascience.com/tdd-datascience-689c98492fcc) it is not impossible to
test even the deep learning model if you try to. But it's not practical at all.

What we have to do is to solve the problem, not to stick to certain algorithm. This fact might not true if you're
working as a researcher so what you want to do is to test your idea, but the most of our cases should focus on the
problem itself rather than the tool which solves them.

Therefore, what we want to measure, and want to be confidence of is the accuracy of the model. We can define
**acceptance test** for it.

All you need to do when you applying your TDD methodology on the development of the machine learning model is
to define your metric to measure the accuracy towards the test data. Define the metric and the threshold of it
becomes a clear goal of your program.

The good things you can earn by doing so is like below.

1. You can documentize the goal accuracy of the model.
2. It force you to divide your model and other codes. This is the advantage comes from the test.
3. You can easily monitor your program since you have metrics on it (e.g. IoT).

The code below is the example with Keras

{% highlight python %}
import pytest
from my.example.ml import MyModel, metric


# You need to prepare you data. Use dependency injection on it.
@pytest.fixture()
def test_data():
    # loads data
    ...
    return test_data


def test_my_model(test_data):
    model = MyModel(...)
    result_predict = map(model.predict, test_data.x)
    result_accuracy = map(metric, zip(test_data.y, result))
    assert result_accuracy > 0.85
{% endhighlight %}

