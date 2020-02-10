---
layout: post
title:  "Simulate a Qubit in Rust"
date:   2020-01-04
tags: [qc, rust]
---

<script type="text/x-mathjax-config">
  MathJax.Hub.Config({
    extensions: [
      "tex2jax.js",  
      "MathMenu.js",
      "MathZoom.js",
      "AssistiveMML.js",
      "a11y/accessibility-menu.js"
    ],
    tex2jax: {      // AND HERE
      inlineMath: [['$', '$']],
      displayMath: [['$$', '$$']]
    },
    jax: ["input/TeX", "output/CommonHTML"],
    TeX: {
      extensions: [
        "AMSmath.js",
        "AMSsymbols.js",
        "noErrors.js",
        "noUndefined.js",
      ]
    }
  });
</script>
<script type="text/javascript" async
  src="https://cdnjs.cloudflare.com/ajax/libs/mathjax/2.7.5/MathJax.js?config=TeX-MML-AM_CHTML">
</script>

The source code I used on this article is upload on my Github ([link](https://github.com/cfr2ak/qubit-rust))

## What is the Qubit?

Qubit is the most fundamental components of the quantum computer. It is same as a bit on a classical one and is called qubit because it is a 'quantum bit'. The state of the qubit is defined as the equation below.

$$|\psi \rangle = \alpha |0\rangle + \beta |1\rangle$$

With this state, when you measure the qubit, you can get the '0' and '1' with the probability of square of alpha and beta on each. Alpha and beta has a relation of equation

$$\alpha^2 + \beta^2 = 1$$

so that make it sure that the total sum of probability becomes 1.

Using the Bloch sphere, we can make this term more expressively.

$$|\psi \rangle = \cos{\frac{\theta}{2}}|0\rangle + e^{i\phi}\sin{\frac{\theta}{2}}|1\rangle$$

Therefore each alpha and beta becomes

$$\alpha = cos{\frac{\theta}{2}}$$

$$\beta = e^{i\phi}\sin{\frac{\theta}{2}}$$

## Implement Qubit in Rust

### Why Rust?

The most difference between the classical bit and the qubit is that the qubit is not able to copy to produce another qubit. But in the most of the programming language, to copy the value of the variable, struct, or the instance of the class is too easy thing to avoid it.

In Rust, there is a paradigm which is called 'ownership'. This makes copying a variable as a nontrivial job. To enable the copy of the variable in Rust, you should implement Copy trait for your struct intentionally. On the other language, you have to implement more code to prohibit copy on a qubit but in Rust, it is default and you even can get compiler error when you make a mistake.

### Create a project

Run the command below to create you project. We will edit main.rs to implement all the code in this article.

{% highlight bash %}
cargo new qubit-rust
{% endhighlight %}

### Implement Qubit type

First, let's define the test for a qubit type.

{% highlight rust %}
#[cfg(test)]
mod tests {
    use super::*;

    #[test]
    fn test_init_qubit() {
        let qubit: Qubit = Default::default();
        assert_eq!(qubit.theta, 0.0);
        assert_eq!(qubit.phi, 0.0);
    }
}
{% endhighlight %}

Then define the qubit and its default value

{% highlight rust %}
struct Qubit {
    theta: f64,
    phi: f64
}

impl Default for Qubit {
    fn default() -> Self {
	    return Qubit{ theta: 0.0, phi: 0.0 };
    }	
}
{% endhighlight %}

Run **cargo test** to check your code works well.

### Implement the measurement

Add a test for measurement. You should get value 0 when its theta has initialized as a 0.0 and 1 when it has initialized as a pi.

{% highlight rust %}
...
    #[test]
    fn test_neasure_default_qubit() {
        let mut qubit: Qubit = Default::default();

        let want = 0;
        let got = Qubit::measure(&mut qubit);
        assert_eq!(got, want);
    }

    #[test]
    fn test_measure_configured_as_one_qubit() {
        let mut qubit = Qubit{ theta: f64::consts::PI, phi: 0.0 };

        let want = 1;
        let got = Qubit::measure(&mut qubit);
        assert_eq!(got, want);
    }
...
{% endhighlight %}

Before writing a implementation, you have to define your dependency on a Cargo.toml as below

{% highlight cargo %}
[dependencies]
rand = "0.7"
{% endhighlight %}

If you finished to add the dependency, add the code below to the top of your main.rs

{% highlight cargo %}
extern crate rand;

use rand::Rng;
{% endhighlight %}

Then implement the measurement.

{% highlight rust %}
...
impl Qubit {
    fn measure(qubit: &mut Qubit) -> i32 {
        let mut rng = rand::thread_rng();
        let rn = rng.gen::<f64>();

        if rn < (qubit.theta / 2.0).cos().powf(2.0) {
            return 0
        }
        return 1
    }
}
...
{% endhighlight %}

Run **cargo test** to check the code works well.

### Implement collapse of the state

Qubit can have a state somewhere between 0 and 1 and can be detected only with that values (it is wrong actually, but let's assume it). Fun part of the qubit is that its state collapse to the state it returned after measurement which means that if you get 0 on the first measurement, you only can get 0 on the further measurement. Let's implement this property to our Qubit type. First, define the test.

{% highlight rust %}
...
    #[test]
    fn test_collapes_of_state() {
        let mut qubit = Qubit{ theta: f64::consts::PI / 2.0, phi: 0.0 };

        let want = Qubit::measure(&mut qubit);

        for _n in 0..100 {
            let got = Qubit::measure(&mut qubit);
            assert_eq!(got, want);
	    }
    }
}
{% endhighlight %}

Then edit measure function to change qubit's state after its measurement.

{% highlight rust %}
...
impl Qubit {
    fn measure(qubit: &mut Qubit) -> i32 {
        let mut rng = rand::thread_rng();
        let rn = rng.gen::<f64>();

        if rn < (qubit.theta / 2.0).cos().powf(2.0) {
            qubit.theta = 0.0;
            return 0
        }
        qubit.theta = f64::consts::PI;
        return 1
    }
}
...
{% endhighlight %}

You've done! You implemented a simulator of a qubit in a Rust. Let's check whether our code works well by changing our main code as below.

### Run and test

{% highlight rust %}
fn main() {
    for _n in 0..1000 {
        let mut qubit = Qubit{ theta: f64::consts::PI / 2.0, phi: 0.0 };
        let result = Qubit::measure(&mut qubit);
        print!("{}", result);
    }
    println!("");
}
{% endhighlight %}

This should prints out random thousand of 0, 1 values.

## What is not implemented here

Actually, there is a phase factor on a equation I've introduced very early of this article but we've never used that. Phase is not able to measure directly but you can apply its relative phase to the state between 0 and 1 with the process using quantum gates. And there is a phenomenon which is called entanglement. Qubit itself is not really interesting since we can get the same result really easily on a classical computer using random seeds too. The entanglement and the phase are the phenomenon which qubit only has while classical one does not. I would like to recommend you to research on your own about these topic.
