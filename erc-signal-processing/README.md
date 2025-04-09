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
title('Noisy Modulated Signal in Time Domain')
```
![Noisy Modulated Signal in Time Domain](https://github.com/omeganebula4/erc-convener-assignment/blob/main/erc-signal-processing/png/Noisy%20Modulated%20Signal%20in%20Time%20Domain.png)

## Second Step: Frequency Domain
The x-axis of the frequency domain is represented by 
```
f = linspace(0, fs, length(y));
```
Then we use the `fft` function to hop into the frequency world.
```
A = abs(fft(y));
```
We plot the magnitude of the complex number representing the frequency amplitude at each point.

```
fig = figure;
plot(f, A);
xlabel('Frequency')
ylabel('Amplitude')
title('Noisy Modulated Signal in Frequency Domain')
```
![Noisy Modulated Signal in Frequency Domain](https://github.com/omeganebula4/erc-convener-assignment/blob/main/erc-signal-processing/png/Noisy%20Modulated%20Signal%20in%20Frequency%20Domain.png)

NOTE: Notice that the `fft` spectrum is symmetric about its middle. This is a consequence of the `fft` function in MATLAB, which plots negative values of frequency at the opposite end, i.e., $f=-f_0$ is now plotted at $f=f_s - f_0$. Also note that `abs(fft(-y)) = abs(fft(y))`, which accounts for the symmetry. This will be important later.


## Third Step: Calculate the Carrier Frequency
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

Since there are only two peaks, they must be symmetric about the carrier frequency. This is because if $I(t) = m(t) \sin (\omega _c t)$ then $\mathcal{F}(I(t)) = \frac{M(\omega - \omega_c) - M(\omega+\omega_c)}{2j}$ where $\mathcal{F}(m(t)) = M(\omega)$. As you can see, $|\mathcal{F} (I(t))|$ is symmetric about $\omega_c$.

Hence, we get 
```
fc = (f(a) + f(b))/2; % Carrier Frequency
```
Carrier frequency for the given audio is approximately 9999.8 Hz.

## Fourth Step: Demodulation
Since the carrier is a sine wave, we set `carrier = sin(2*pi*fc*t);`.

Now, the question remains, why do we multiply the message with the carrier again for demodulation.

Let $I(t) = m(t) \sin (\omega_c t)$ be the modulated signal.
Note that, there is no way to extract $m(t)$ directly. You cannot divide $I(t)$ with $\sin(\omega_c t)$ since the carrier frequency is comparable to the sampling frequency, so we would be dividing by zero most of the time.

However, if we multiply $I(t)$ with $\sin(\omega_c t)$ again, it gives $I(t) \sin(\omega_c t) = m(t) \sin^2 (\omega_c t) = \frac{m(t)}{2} - \frac{m(t) \cos(4 \omega_c t)}{2}$. We can eliminate the high frequency component $\frac{m(t) \cos(4 \omega_c t)}{2}$ by passing $I(t)$ through a low pass filter, which gives us with a demodulated message signal.

```
carrier = sin(2*pi*fc*t);
square = y .* carrier';

demod_y = 2*lowpass(square, fc/2, fs);
```
This code retreives $m(t)$ as `demod_y`, also maintaining the amplitude. Note that the cutoff frequency for the low-pass filter is kinda arbitrary (I think). Too high of a cutoff frequency would allow the undesired high frequency components to pass through as well.

Again, we plot this graph in time domain.
```
fig = figure;
plot(t, demod_y);
xlabel('Time')
ylabel('Intensity')
title('Noisy Demodulated Signal in Time Domain')
```
![Noisy Demodulated Signal in Time Domain](https://github.com/omeganebula4/erc-convener-assignment/blob/main/erc-signal-processing/png/Noisy%20Demodulated%20Signal%20in%20Time%20Domain.png)


## Fifth Step: Domain Expansion: Back to Frequency
This is pretty straight-forward. We take the magnitude of fast fourier transform `fft` on the demodulated function from fourth step.
```
demod_A = abs(fft(demod_y));
```
Then we plot this graph
```
fig = figure;
plot(f, demod_A);
xlabel('Frequency')
ylabel('Amplitude')
title('Noisy Demodulated Signal in Frequency Domain')
```
![Noisy Demodulated Signal in Frequency Domain](https://github.com/omeganebula4/erc-convener-assignment/blob/main/erc-signal-processing/png/Noisy%20Demodulated%20Signal%20in%20Frequency%20Domain.png)

## Sixth Step: Calculating Bandwidth
We set a quality factor $Q$ which basically says the bandwidth should be calculated as the difference in frequencies of the two points for which amplitude = `max_amp/Q`.

We set $Q$ and find the `max_amp` and the index at which it occurs in the array.
```
Q = sqrt(3); % Qaulity Factor
[max_amp, max_amp_index] = max(demod_A(1:round(length(y)/2)));
new_arr = abs(demod_A - max_amp/Q); % new array to find f_L and f_H
```
Again, we only consider the first half of the array `demod_A`. We just need to find the frequencies/indices at which minima of `new_arr` occurs at either side of the peak achieved at `max_amp_index`.
```
[minL, min_lower_index] = min(new_arr(1:max_amp_index));
[minH, min_higher_index] = min(new_arr(max_amp_index:round(length(y)/2)));
```
Then we find the frequencies $f_L$ and $f_H$ using these indices.
```
fl = f(min_lower_index);
fh = f(max_amp_index+min_higher_index-1);
```

## Final Step: Filtering
we use a bandpass filter on the demodulated wave (in time domain), with our newly acquired bandwidth frequencies.
```
demod_denoise_y = bandpass(demod_y, [fl, fh], fs);
```
Then we plot this graph and admire its beauty.
```
fig = figure;
plot(t, demod_denoise_y);
xlabel('Time')
ylabel('Intensity')
ylim([-1 1])
title('Filtered Demodulated Signal in Time Domain')
```
![Filtered Demodulated Signal in Time Domain](https://github.com/omeganebula4/erc-convener-assignment/blob/main/erc-signal-processing/png/Filtered%20Demodulated%20Signal%20in%20Time%20Domain.png)

We also plot this graph in frequency domain just because we can.
```
demod_denoise_amp = abs(fft(demod_denoise_y));

fig = figure;
plot(f, demod_denoise_amp);
xlabel('Frequency')
ylabel('Amplitude')
title('Filtered Demodulated Signal in Frequency Domain')
```
![Filtered Demodulated Signal in Frequency Domain](https://github.com/omeganebula4/erc-convener-assignment/blob/main/erc-signal-processing/png/Filtered%20Demodulated%20Signal%20in%20Frequency%20Domain.png)

## Comparision
We have finally freed the audio file from the cluches of chaos. Here's the difference
### Before
![Noisy Modulated Signal in Time Domain](https://github.com/omeganebula4/erc-convener-assignment/blob/main/erc-signal-processing/png/Noisy%20Modulated%20Signal%20in%20Time%20Domain.png)

### After
![Filtered Demodulated Signal in Time Domain](https://github.com/omeganebula4/erc-convener-assignment/blob/main/erc-signal-processing/png/Filtered%20Demodulated%20Signal%20in%20Time%20Domain.png)


![Here's](https://github.com/omeganebula4/erc-convener-assignment/blob/main/erc-signal-processing/demodulated_filtered_audio.wav) the final audio file as well.

Also ![here](https://github.com/omeganebula4/erc-convener-assignment/blob/main/erc-signal-processing/pdf/signal_process_erc.pdf) is the published MATLAB code with all figures.