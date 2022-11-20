Using a suggestion from Daniel Estévez I've implemented a flow graph to study the viability of this method to extract Doppler data from Artemis 1's OQPSK modulation.  As my antenna is too small to demodulate the signal to obtain Doppler data we are trying different techniques to extract useful Doppler data from the signal.

I made a recording of Artemis 1 while near earth and it was sending idle data.  This can be found here:
https://www.dropbox.com/s/wjft1ece7rzjdfl/gqrx_20221116_142706_2216499998_4000000_fc.raw?dl=0


The flow graph uses a frequency xlating FIR block to decimate and potentially shift the output as needed.  Next, per Daniel's suggestion the output is raised to the 4th power with a multiply block with four inputs of the same output from the same signal.

![My Image](https://github.com/ScottTilley/Artemis1/blob/main/RnD/OQPSK_Doppler_extract2.png)

Frequency plot view of the results of the flow graph with the semi-raw signal upper (BW truncated by the frequency xlating FIR block to limit interefering signals) and the bottom being the output from the multiply block producing a CW carrier at 4x the Doppler frequency,

![My Image](https://github.com/ScottTilley/Artemis1/blob/main/RnD/OQPSK_doppler_extract1.png)

Finally I ran the flowgraph and outputed the data repeating from the file above and perform an FFT with STRF and ploted the results (1s x 1Hz bins).  You can see the increasing Doppler from Artemis 1 just before the first TCM burn.  I entered an arbitrary frequency as I believe this is 4x the frequency so some work is needed to properly interprete the data.  

![My Image](https://github.com/ScottTilley/Artemis1/blob/main/RnD/OQPSK_doppler_extract3.png)

The thing that is 4x is the baseband frequency. Say that after the multiplication block the CW carrier appears at 10 kHz in the baseband. Then this means the the centre of the OQPSK signal in the baseband was actually 2.5 kHz (10 kHz / 4). If you were tuned to 2261.5 MHz, then the exact frequency would be 2261.5 MHz + 2.5 kHz. Probably you see what I mean with this example.

A nice trick with STRF is to pretend that you're recording at 4x your centre frequency (so if the SDR is actually tuned to 2216.5 MHz you would lie to STRF and tell it that you're recording at 8866 MHz). You would also lie with the nominal transmit frequency, and say that this spacecraft transmits at 8866 MHz. Then what happens is that if you tell STRF to plot Doppler traces corresponding to TLEs on top of your waterfall, the traces correctly match your data, as you have accounted for the fact that "there is a 4x zoom in the frequency".

Indeed this multiply trick doesn't work very well on low SNR. There are so called "squaring losses", which in this case are proportional to SNR^4, because we are using the 4-th power (with BPSK we would use the 2-nd power, and get losses proportional to SNR^2). This means that things become much worse as the SNR decreases.

To make things worse, all of what I said applies verbatim to regular BPSK or QPSK. OQPSK is an odd animal. In general the 4-th power trick will behave worse (give a CW carrier with worse SNR) than for regular QPSK. How worse depends on the pulse shape (which still I haven't measured for Orion). For MSK (which is OQPSK with half-sine pulses) the 4-th power trick wouldn't work. There wouldn't be any CW carrier. But Orion uses a different pulse shape, so we get a CW carrier. It doesn't seem to be an RRC pulse either, as recommended by CCSDS for Category A spacecraft. 

Still I think that the 4-th power trick will give you a CW carrier with more SNR than if you were to pick one of the spectral lines of the signal (specially because these are only strong when idle frames are transmitted; when there is useful data, the spectral lines are very weak because the data is encrypted).

An alternative method for measuring Doppler with weak SNR would be to correlate against known pieces of the signal. We could use the 64-bit ASM, perhaps followed by part of the header (spacecraft ID and so on are known and fixed), but that has short duration.

Idle frames are interesting, because their contents are ~980 bits of 0x00, 0x01, 0x02, 0x03, etc. padding bytes. Then we have 1024 bits of LDPC parity bits which at first glance look pretty much random due to counters changing in the header (but maybe they are not so random; a look a the Tanner graph would give more info).

The problem is that idle frames only happen sometimes, so I'm not so keen relying on them to measure Doppler.
