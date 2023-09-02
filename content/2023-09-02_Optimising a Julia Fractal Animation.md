I recently came across [this video](https://youtu.be/g4vN2Z0JuZI?si=50XyIeCj0LvA5puI) on YouTube. It's a short tutorial on generating a Julia set fractal image in Rust. I've used rust before but never with image generation or mathematics, so it looked interesting and I followed along. By then end of the video you have some code to generate a single black and white image.

I extended the code to use colour and generate an animation.

# How does it work?

1. define an iterative function, say `z = z² + c`
2. take a point on the complex plane `z = x + yi` and a constant `c`
3. iterate until the value diverges to "infinity", in this case until the magnitude exceeds 2
4. stop if exceeding `MAX_ITER`
5. the Julia set is the set of points that don't diverge
6. for the points that do diverge, we record and colour based on the number of iterations it took to diverge.

In the tutorial, he performed 255 iterations, and mapped that to grayscale.

I extended it to an arbitrary number of iterations and mapped the iterations to the hue component in HSV colour. I also used it for the value component, to darken the earlier iterations. This was a stylistic choice to prevent the images from being too bright red.

I ran it with 1000+ iterations and increased the resolution to 1080p, and the result was pretty nice.

## Animation

My next idea was to animate it. I wanted to see how the image looks "over time". Each frame would use `MAX_ITER` corresponding the frame number. In the basic first version, all it did was vary the `MAX_ITER` and generate each image independently in sequence. This turned out to be very slow, taking 2210 seconds (~37 minutes) for 360 frames. That is a long time for a 15 second video (24fps).

I also timed each frame:
<iframe width="600" height="371" seamless frameborder="0" scrolling="no" src="https://docs.google.com/spreadsheets/d/e/2PACX-1vTFf5LQnajqj0EReuj4TshAeTfe3XIF9aJUBV6LPWKiSrJ_QDXR_Fc_WjVI5T8B3yFNplbLbYFbFMbW/pubchart?oid=813755211&amp;format=interactive"></iframe>
You can see that as `MAX_ITER` increases, the frame time increases, but the rate of increase slows, because less and less points (pixels) require the increased iterations.

# Optimisation

## Multithreading

The first version used only a single thread, but my CPU has 12 threads, so I decided to use multithreading to calculate the frames in parallel. I programmed it to use one less than the number of available threads, allowing 1 thread for the rest of the applications running on my PC. In a way, I "turned it up to 11"!

```rust
use std::thread;

fn num_threads() -> usize {
    thread::available_parallelism().unwrap().get() - 1
}
```

I initially divided the frames into separate lists for each thread, but later I switched to a multi consumer message passing pattern using the `crossbeam_channel` package. I'm not sure if there was any significant performance improvement but the code looked cleaner. The latter pattern would allow for threads to pick up the slack if other threads are slower, though in this case the load should be pretty even across threads.

With multithreading, the total time reduced to 318 seconds (~5 minutes). This is a significant improvement!
<iframe width="687" height="425" seamless frameborder="0" scrolling="no" src="https://docs.google.com/spreadsheets/d/e/2PACX-1vTFf5LQnajqj0EReuj4TshAeTfe3XIF9aJUBV6LPWKiSrJ_QDXR_Fc_WjVI5T8B3yFNplbLbYFbFMbW/pubchart?oid=1710772616&amp;format=interactive"></iframe>

However, it is not 11 times the improvement, and you can see in the frame times that each frame takes longer. This is probably due to CPU contention of the threads, and maybe i should have used less than 11 and left more for the other applications. It could also be contention related to the disk, when saving the images in parallel. Regardless, I stuck with 11 for the remainder.
<iframe width="613" height="380" seamless frameborder="0" scrolling="no" src="https://docs.google.com/spreadsheets/d/e/2PACX-1vTFf5LQnajqj0EReuj4TshAeTfe3XIF9aJUBV6LPWKiSrJ_QDXR_Fc_WjVI5T8B3yFNplbLbYFbFMbW/pubchart?oid=1922109242&amp;format=interactive"></iframe>

From here on I switched to generating 1000 frames (~42 seconds). With multithreading this took 1066 seconds (~18 minutes).

## Hashmap Cache

My next insight, was that in each new frame, I was recalculating every pixel, even if it had already diverged in previous frames. I could reduce the calculations if I remember those results and carry them forward in to the following frames. I.E. if in frame 50 we know that a certain point diverges on the 50th iteration, I can cache that value of 50 for those coordinates, and in later frames I can first check the cache if a value was already calculated. In many cases the natural cache is a hashmap/dictionary. For any hashable key you can store a value, and later retrieve the corresponding value if it exists.

Each thread get's it's own independent cache, so there are some repeated calculations as each thread calculates each pixel regardless of if it was cached in the other thread. I did try to use a shared cache, but using a mutex added significant cost due to threads waiting for the lock on the cache before adding and retrieving values, that it became slower than having independent caches.

The hashmap cache reduced the total time to 514 seconds (~8.5 minutes). Also the curve of the frame times looks very different:
<iframe width="613" height="380" seamless frameborder="0" scrolling="no" src="https://docs.google.com/spreadsheets/d/e/2PACX-1vTFf5LQnajqj0EReuj4TshAeTfe3XIF9aJUBV6LPWKiSrJ_QDXR_Fc_WjVI5T8B3yFNplbLbYFbFMbW/pubchart?oid=1551625327&amp;format=interactive"></iframe>
Initially the frame times are higher, due to the costs of checking the cache, adding to the cache. Most of the cache additions are likely to be early on. Later the times are lower because it is starting to use the cache, and eventually the frame times start reducing as it starts using the cache more and more. Eventually it flattens, as the majority of pixels are from the cache and retrieving from the cache is constant in time.

## Matrix Cache

The initial costs were high with the hashmap. This is because getting and setting with the hashmap, you need to hash the value, and in this case that is actually a waste of time. The keys in this case are coordinates, so a 2D array is a much more better option for a cache. I used `DMatrix` from the `nalgebra` package. This reduced the total time to 426 seconds (~7 minutes), and you can see in the frame times that the curve is the same shape (since the function of the cache is the same), but it is exclusively lower than the hashmap times, especially at the start.
<iframe width="613" height="380" seamless frameborder="0" scrolling="no" src="https://docs.google.com/spreadsheets/d/e/2PACX-1vTFf5LQnajqj0EReuj4TshAeTfe3XIF9aJUBV6LPWKiSrJ_QDXR_Fc_WjVI5T8B3yFNplbLbYFbFMbW/pubchart?oid=511978215&amp;format=interactive"></iframe>

## Max First

At this point I was saving time on pixels that diverged, but it felt like a waste to start from scratch every time for the pixels that had not diverged yet. For example if a pixel had not diverged after 49 iterations in frame 49, it would start again and do 50 iterations, then 51 more, then 52 more etc. I thought about storing the current progress in another cache and passing that through the threads. This would need to store the current value of z along with the number of iterations that got it there, rather than just the number of iterations to divergence like the existing cache. However this complicates the calculation logic quite a bit.

I then had the realization, that I could simply calculate the final frame with the `MAX_ITER` set to the largest value first, and then derive all the other frames from it. This final frame would would be a matrix with values from 0 to `MAX_ITER` and to generate the the `n`th frame I would simply disregard any pixels which had a value greater than `n`.

This method reduced the total time to 302 seconds (~5 minutes). And now you can see the frame times are pretty much constant! In a single thread it calulates the final frame matrix, taking 8 seconds, then 294 seconds to generate all the images.
<iframe width="613" height="380" seamless frameborder="0" scrolling="no" src="https://docs.google.com/spreadsheets/d/e/2PACX-1vTFf5LQnajqj0EReuj4TshAeTfe3XIF9aJUBV6LPWKiSrJ_QDXR_Fc_WjVI5T8B3yFNplbLbYFbFMbW/pubchart?oid=416816937&amp;format=interactive"></iframe>

## Parallel Pixels

A small optimisation I then did was to calculate the pixels of the final frame matrix in parallel. I again used message passing. This reduced the initial time from 8 seconds to 1.56 seconds.
<iframe width="687" height="425" seamless frameborder="0" scrolling="no" src="https://docs.google.com/spreadsheets/d/e/2PACX-1vTFf5LQnajqj0EReuj4TshAeTfe3XIF9aJUBV6LPWKiSrJ_QDXR_Fc_WjVI5T8B3yFNplbLbYFbFMbW/pubchart?oid=119253201&amp;format=interactive"></iframe>

The frame times are of course pretty much the same as before.
<iframe width="600" height="371" seamless frameborder="0" scrolling="no" src="https://docs.google.com/spreadsheets/d/e/2PACX-1vTFf5LQnajqj0EReuj4TshAeTfe3XIF9aJUBV6LPWKiSrJ_QDXR_Fc_WjVI5T8B3yFNplbLbYFbFMbW/pubchart?oid=1561861481&amp;format=interactive"></iframe>

## Image Formats

As you can see in the previous chart, the frame times are now approximately constant at 3.2 seconds. This still felt slow, and I wondered if this was related to the compression used in generating PNG images. I tried JPG and BMP formats, and the difference was huge.
<iframe width="613" height="380" seamless frameborder="0" scrolling="no" src="https://docs.google.com/spreadsheets/d/e/2PACX-1vTFf5LQnajqj0EReuj4TshAeTfe3XIF9aJUBV6LPWKiSrJ_QDXR_Fc_WjVI5T8B3yFNplbLbYFbFMbW/pubchart?oid=898139394&amp;format=interactive"></iframe>
BMP is around 5 times faster than PNG! This makes sense, as BMP is the simplest image format with no compression. With BMP the total time is only 62 seconds (~1 minute)!

Though the drawback is the filesize, with each 1080p frame being 5.93MB. A single 1000 frame animation would be 6 GB. At one point I filled up my whole disk with frames for the final animations! I was playing with higher resolutions and around 10000 iterations. I found that images don't change much at iterations higher than around 1000 so I reduced it after that.

# More Charts

Here are some charts with all the data:
<iframe width="613" height="380" seamless frameborder="0" scrolling="no" src="https://docs.google.com/spreadsheets/d/e/2PACX-1vTFf5LQnajqj0EReuj4TshAeTfe3XIF9aJUBV6LPWKiSrJ_QDXR_Fc_WjVI5T8B3yFNplbLbYFbFMbW/pubchart?oid=1055457345&amp;format=interactive"></iframe>
<iframe width="687" height="425" seamless frameborder="0" scrolling="no" src="https://docs.google.com/spreadsheets/d/e/2PACX-1vTFf5LQnajqj0EReuj4TshAeTfe3XIF9aJUBV6LPWKiSrJ_QDXR_Fc_WjVI5T8B3yFNplbLbYFbFMbW/pubchart?oid=1131226907&amp;format=interactive"></iframe>
<iframe width="687" height="425" seamless frameborder="0" scrolling="no" src="https://docs.google.com/spreadsheets/d/e/2PACX-1vTFf5LQnajqj0EReuj4TshAeTfe3XIF9aJUBV6LPWKiSrJ_QDXR_Fc_WjVI5T8B3yFNplbLbYFbFMbW/pubchart?oid=135305384&amp;format=interactive"></iframe>

# Final Video

For the final video I generated animations for a number of parameter variations. I used `ffmpeg` to stitch the BMP sequences into MP4 videos, then I used Davinci Resolve to edit them together. The animations turned out better than I imagined. Because the colour scale is adjusted to each frame independently, there is a feeling of motion as the colours "follow" the iterations. I'm very happy with the result.

The final video is [here](https://youtu.be/oNpR1SvzMXE) and the source code can be found [here](https://github.com/skarfie123/julia).

<iframe width="560" height="315" src="https://www.youtube-nocookie.com/embed/oNpR1SvzMXE?si=T5EzRsu-Jw9FG-Oa" title="YouTube video player" frameborder="0" allow="accelerometer; autoplay; clipboard-write; encrypted-media; gyroscope; picture-in-picture; web-share" allowfullscreen></iframe>
