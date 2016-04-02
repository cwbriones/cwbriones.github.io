---
layout: post
date: 2016-03-31 18:14:44
title: Up and Running with Jekyll
published: true
---

Here's that famous quote from hipster ipsum:

>Actually deep v letterpress, High Life semiotics viral umami Godard YOLO +1
>chillwave retro banjo sartorial farm-to-table. Mumblecore pug selvage Banksy +1
>chambray, Odd Future blog butcher lomo. Chillwave viral gluten-free wolf Austin,
>selvage 90's ennui skateboard. Trust fund letterpress hella yr synth leggings,
>skateboard literally blog salvia jean shorts forage Terry Richardson wolf food
>truck. Butcher gastropub cred, American Apparel Tonx McSweeney's kogi distillery
>slow-carb twee. Meggings Brooklyn selfies Pinterest gastropub. Readymade Terry
>Richardson kale chips mustache 8-bit YOLO, Marfa slow-carb deep v
>wayfarers DIY iPhone Carles.

And now for some code:

{% highlight rust %}
extern crate rand;

use rand::{thread_rng, Rng};

fn partition<T: Num>(items: &mut [T], lo: usize, hi: usize) {
    // Choose the pivot
    let mut rng = thread_rng();
    let pivot_idx = rng.gen_range(lo, hi + 1);
    swap(lo, pivot_idx);

    // Partition
    let left = lo;
    let right = hi;
    loop {
        while (items[left] < items[lo] && left < hi) {
          left += 1;
        }
        while (items[lo] < items[right] && right > lo) {
          right -= 1;
        }
        if (right <= left) {
            break;
        }
        items.swap(right, left);
    }
    items.swap(right, lo);
    return right;
}
{% endhighlight %}

I also could have put a small piece here like this: `$ echo Hello World!`.

As well as lists...

* One
* and a two
* and a three

and style some text!

*this is emphasized*

**this is bold**

~~this was redacted.~~

Finally I'll link to my [home page](/) for you.
