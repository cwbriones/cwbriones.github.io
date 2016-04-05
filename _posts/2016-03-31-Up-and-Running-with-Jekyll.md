---
layout: post
date: 2016-03-31 18:14:44
title: Up and Running with Jekyll
published: true
---

Here's that famous quote from hipster ipsum:[^1]

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

{% highlight elixir %}
defmodule Example do
  def run_command("frobnicate", post) do
    size = String.length(post)
    String.duplicate(post, size)
  end
end
{% endhighlight %}

{% highlight rust %}
extern crate rand;

use rand::{thread_rng, Rng};

///  The entire contents of this comment happen to be exactly eighty lines long.

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

# Other Markdown

I also could have put a small piece here like this: `$ echo Hello World!`.

As well as lists...[^2]

* One
* and a two
* and a three

And we can style some text! *This is emphasized.* **This is bold.** ~~This was redacted.~~

| A | messy | table|
|--:|:-----:|:-----|
|still|looks|nice|

Finally I'll link to my [home page](/) for you.

[^1]: With some fancy footnote!
[^2]: These are just unordered lists, you can also have ordered ones.
