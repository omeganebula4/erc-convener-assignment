# Freeing the Audio
## First Step: Capture the audio
First things first, we import the audio into our program.
```
[y, fs] = audioread("modulated_noisy_audio.wav");
```
`fs` is the sampling frequency, meaning a new value of the intensity `y` was recorded after every time interval $t = \frac{1}{f_s}$.

Hence, our time interval would be
```
t = linspace(0, length(y)/fs, length(y));
```
Then, we simply plot the graph for intensity vs time of the audio file.
```
fig = figure;
plot(t, y);
xlabel('Time')
ylabel('Intensity')
ylim([-1.2 1.2])
title('Noisy Signal in Time Domain')
```
![Noisy Signal in Time Domain](https://github.com/omeganebula4/erc-convener-assignment/blob/main/erc-signal-processing/png/Noisy%20Signal%20in%20Time%20Domain.png)

## Second Step: Frequency Domain
The x-axis of the frequency domain is represented by 
```
f = linspace(0, fs, length(y));
```
Then we use the `fft` function to hop into the frequency world.
```
fourier_t = fft(y);
A = abs(fourier_t);
```
We plot the magnitude of the complex number representing the frequency amplitude at each point.

```
fig = figure;
plot(f, A);
xlabel('Frequency')
ylabel('Amplitude')
title('Noisy Signal in Frequency Domain')
```
![Noisy Signal in Frequency Domain](https://github.com/omeganebula4/erc-convener-assignment/blob/main/erc-signal-processing/png/Noisy%20Signal%20in%20Frequency%20Domain.png)

NOTE: Notice that the `fft` spectrum is symmetric about its middle. This is a consequence of the `fft` function in MATLAB, which plots negative values of frequency at the opposite end, i.e., $f=-f_0$ is now plotted at $f=f_s - f_0$. Also note that `abs(fft(-y)) = abs(fft(y))`, which accounts for the symmetry. This will be important later.

## Third Step: Eliminate the Noise
We set a threshold for the amplitude of the audio in frequency domain `min_A = max(A)/2`. We will eliminate every point in `A` (and the corresponding point in `fourier_t`) with amplitude less than `min_A`.
```
min_A = max(A)/2;
denoise_fourier_t = fourier_t .* (A>min_A);
denoise_A = abs(denoise_fourier_t);
```
Now we have new variables encapsulating the noise-free frequency domain representation of the audio.

We plot this graph.
```
fig = figure;
plot(f, denoise_A);
xlabel('Frequency')
ylabel('Amplitude')
title('Filtered Signal in Frequency Domain')
```
![Filtered Signal in Frequency Domain](https://github.com/omeganebula4/erc-convener-assignment/blob/main/erc-signal-processing/png/Filtered%20Signal%20in%20Frequency%20Domain.png)

## Fourth Step: Calculate the Carrier Frequency (and the message frequency)
Now comes the confusing part. Note that since the plot is symmetric about its middle (as explained at the end of second step), it suffices to calculate the carrier and message frequency using the first two peaks (in increasing order of frequency). Hence, we only consider the peaks in that region.

```
[pks, indices] = findpeaks(denoise_A(1:round(length(y)/2)));
```

Note that `indices` is the array of indices of `denoise_A` which correspond to the element of `pks`, i.e., `denoise_A(indices(i)) = pks(i)`.

Now, MATLAB gives us a lot of irrelevant peaks which we need to comb through. We first order `pks` in descending order, then shuffle `indices` to match them.
```
[pks, I] = sort(pks, "descend");
indices = indices(I);
```
Now, note that since `f = linspace(0, fs, length(y));` is linear with index, a higher element of `indices` corresponds to a higher `f`. 

Now, to get a relevant peak, we say that the 2nd highest peak (which we want) should be a considerable distance away from the 1st peak, and this "distance" is proportional to the difference `|indices(i) - indices(1)|`.

So, we loop through the "sorted" (according to descending order of `pks`) and keep increasing the index until we find the highest element which has a bigger index difference than 2200 (no sense to this value, its a pilot reading, kind of).

This index corresponds to the second peak of amplitude in frequency domain.

Now, since `indices[i]` was proportional to the frequency, we find the index of the peak with higher frequency and the peak with lower frequency.
```
a = max([indices(1) indices(looper)]); % index for larger frequency
b = min([indices(1) indices(looper)]); % index for lower frequency
```

Since there are only two peaks, they must be symmetric about the carrier frequency. We can also derive it using $\cos(2 \pi f_c t) \cos (2\pi f_m t)) = \frac{1}{2} (\cos(2\pi (f_c + f_m)t) + \cos(2\pi (f_c - f_m)t)$.
So the Fourier transform would peak at $f_c + f_m = f(a)$ and $f_c - f_m = f(b)$.

Hence, we get 
```
fc = (f(a) + f(b))/2; % Carrier Frequency
fm = (f(a) - f(b))/2; % Message Frequency
```
Carrier frequency for the given audio is approximately 9999.8 Hz.

## Fifth Step: Domain Expansion: Back to Time
This is pretty straight-forward. We take the inverse fast fourier transform `ifft` on the filtered function from third step.
```
denoise_y = ifft(denoise_fourier_t);
```
Then we plot this graph
```
fig = figure;
plot(t, denoise_y);
xlabel('Time')
ylabel('Intensity')
ylim([-0.8 0.8])
title('Filtered Signal in Time Domain')
```
![Filtered Signal in Time Domain](https://github.com/omeganebula4/erc-convener-assignment/blob/main/erc-signal-processing/png/Filtered%20Signal%20in%20Time%20Domain.png)

## Final Step: Demodulation
We use the ![amdemod](https://in.mathworks.com/help/comm/ref/amdemod.html) function from the Communications Toolbox in MATLAB.
```
denoise_demod_y = amdemod(denoise_y, fc, fs);
```
And again, we plot the graph like good ol' times.
```
fig = figure;
plot(t, denoise_demod_y);
xlabel('Time')
ylabel('Intensity')
ylim([-0.8 0.8])
title('Filtered Demodulated Signal in Time Domain')
```
![Filtered Demodulated Signal in Time Domain](https://github.com/omeganebula4/erc-convener-assignment/blob/main/erc-signal-processing/png/Filtered%20Demodulated%20Signal%20in%20Time%20Domain.png)

## Comparision
We have finally freed the audio file from the cluches of chaos. Here's the difference
### Before
![Noisy Signal in Time Domain](https://github.com/omeganebula4/erc-convener-assignment/blob/main/erc-signal-processing/png/Noisy%20Signal%20in%20Time%20Domain.png)

### After
![Filtered Demodulated Signal in Time Domain](https://github.com/omeganebula4/erc-convener-assignment/blob/main/erc-signal-processing/png/Filtered%20Demodulated%20Signal%20in%20Time%20Domain.png)


![Here's](https://github.com/omeganebula4/erc-convener-assignment/blob/main/erc-signal-processing/demodulated_filtered_audio.wav) the final audio file as well.