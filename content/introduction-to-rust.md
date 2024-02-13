+++
title = "A gentle introduction to checking code correctness in Verus"
slug = "verus-gentle-introduction"
author = "Andrea Lattuada"

[taxonomies]
authors = ["Andrea Lattuada"]
+++


<style type="text/css">
.note {
    font-size: 80%;
    line-height: 1em;
}
</style>

I'm going to talk about the tool we've been building, called Verus, which is a tool to verify Rust, and specifically to verify Rust code in the domain of system software: operating systems, file systems, things like that.

<!-- more -->

<i class="note">This post is based, in part, on the talk ["Verified Rust for low-level systems code - Rust Zürisee June 2023"](https://www.youtube.com/watch?v=ZZTk-zS4ZCY).</i>

### Checking code correctness with testing

Instead of trying to explain what verification is, I'm just going to jump into an example. You're writing a very simple max function here. What you might do is try to write a test for it. 
```rust,linenos
fn max(a: u64, b: u64) -> u64 {
    if a >= b { b } else { a }
}

fn max_test_1() {
    let x = 4;
    let y = 4;
    let ret = max(x, y);
    assert!(ret == 4);
}

fn main() {
    max_test_1();
    println!("success");
}
```

I'm keeping things extremely simple here, just writing a simple test. We have a max function, we call it with certain arguments, and check that the result is the one we expect. If we run this program, it passes. We wrote a test, it passes, we're happy.

You may have already spotted the problem here. Two problems, in fact: there's a bug, but also that the test case I wrote was insufficient. It was testing a very specific case and didn't catch the "edge" case. You could go and write another test and run it, and now finally, we get the test failing because we swapped DNA over here.

```rust,linenos
fn max_test_2() {
    let x = 4;
    let y = 3;
    let ret = max(x, y);
    assert_eq!(ret, 4);
}

fn main() {
    max_test_1();
    max_test_2();
    println!("success");
}
```

That's how test-based development works, and it can work well. One needs to pay attention that the tests they write will catch edge cases or unexpected behavior. Let's see if we can do better: the way I'm writing this next test is a bit different. Instead of writing the return value that we expect exactly, I'm stating the property I want the return value to have.

```rust,linenos
fn max_test_3() {
    let x = 4;
    let y = 3;
    let ret = max(x, y);
    assert!(ret == x || ret == y);
    assert!(ret >= x && ret >= y);
}

fn main() {
    max_test_3();
    println!("success");
}
```

I'm saying the return value here should be either of the two arguments and should be the greater one of the two. In this test we would fail the assertion on line 6, because `max` is not returning the maximum.

When writing tests you have to write a test case for each point in the input space of this function, but you never really know that you've caught all the possible issues and bugs because you can't possibly test for all the cases.

### From tests to formal specifications

To use our tool, Verus, we need to re-organize the code a bit.

```verus,linenos
use vstd::prelude::*;
verus! {

fn max(a: u64, b: u64) -> u64 {
    if a >= b { a } else { b }
}

fn max_test() {
    let x = 4;
    let y = 3;
    let ret = max(x, y);
    assert(ret == x || ret == y);
    assert(ret >= x && ret >= y);
}

} // verus!
```

I took the max function from before, fixed the issue, and then wrote Verus assertions as part of the `max_test` functions (on lines 12 and 13): these look different from regular rust assertion but I'm keeping the property that we wanted: the return value is either of the inputs and it is greater or equal to both.

<pre class="ansi-html"><code><span style="font-weight:bold;color:#8e8e8e;"></span><span style="font-weight:bold;color:#ffc4bd;">error</span><span style="font-weight:bold;color:#8e8e8e;">: assertion failed</span>
  <span style="font-weight:bold;color:#8e8e8e;"></span><span style="font-weight:bold;color:#c1e3fe;">--&gt; </span>/Users/andreal1/Demo/A2-program-0.rs:13:12
   <span style="font-weight:bold;color:#8e8e8e;"></span><span style="font-weight:bold;color:#c1e3fe;">|</span>
<span style="font-weight:bold;color:#8e8e8e;"></span><span style="font-weight:bold;color:#c1e3fe;">13</span> <span style="font-weight:bold;color:#8e8e8e;"></span><span style="font-weight:bold;color:#c1e3fe;">|</span>     assert(ret == x || ret == y);
   <span style="font-weight:bold;color:#8e8e8e;"></span><span style="font-weight:bold;color:#c1e3fe;">| </span>           <span style="font-weight:bold;color:#8e8e8e;"></span><span style="font-weight:bold;color:#ffc4bd;">^^^^^^^^^^^^^^^^^^^^</span> <span style="font-weight:bold;color:#8e8e8e;"></span><span style="font-weight:bold;color:#ffc4bd;">assertion failed</span>

<span style="font-weight:bold;color:#8e8e8e;"></span><span style="font-weight:bold;color:#ffc4bd;">error</span><span style="font-weight:bold;color:#8e8e8e;">: assertion failed</span>
  <span style="font-weight:bold;color:#8e8e8e;"></span><span style="font-weight:bold;color:#c1e3fe;">--&gt; </span>/Users/andreal1/Demo/A2-program-0.rs:14:12
   <span style="font-weight:bold;color:#8e8e8e;"></span><span style="font-weight:bold;color:#c1e3fe;">|</span>
<span style="font-weight:bold;color:#8e8e8e;"></span><span style="font-weight:bold;color:#c1e3fe;">14</span> <span style="font-weight:bold;color:#8e8e8e;"></span><span style="font-weight:bold;color:#c1e3fe;">|</span>     assert(ret &gt;= x &amp;&amp; ret &gt;= y);
   <span style="font-weight:bold;color:#8e8e8e;"></span><span style="font-weight:bold;color:#c1e3fe;">| </span>           <span style="font-weight:bold;color:#8e8e8e;"></span><span style="font-weight:bold;color:#ffc4bd;">^^^^^^^^^^^^^^^^^^^^</span> <span style="font-weight:bold;color:#8e8e8e;"></span><span style="font-weight:bold;color:#ffc4bd;">assertion failed</span>

<span style="font-weight:bold;color:#8e8e8e;"></span><span style="font-weight:bold;color:#ffc4bd;">error</span><span style="font-weight:bold;color:#8e8e8e;">: aborting due to 2 previous errors</span>

verification results:: 1 verified, 1 errors
</code></pre>

The error message we get from Verus is different; it's not a panic. We actually never run the code here. What Verus does is it statically checks every possible way that the program could run and checks whether these assertions are true in every possible case.

<i>...</i>
