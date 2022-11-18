Using a suggestion from Daniel Estévez I've implemented a flow graph to study the viability of this method to extract Doppler data from Artemis 1's OQPSK modulation.  As my antenna is too small to demodulate the signal to obtain Doppler data we are trying different techniques to extract useful Doppler data from the signal.

I made a recording of Artemis 1 while near earth and it was sending idle data.  This can be found here:


The flow graph uses a frequency xlating FIR block to decimate and potentially shift the output as needed.  Next, per Daniel's suggestion the output is raised to the 4th power with a multiply block with four inputs of the same output from the same signal.

[My Image](https://github.com/ScottTilley/Artemis1/blob/main/RnD/OQPSK_Doppler_extract2.png)

Frequency plot view of the results of the flow graph with the semi-raw signal upper (BW truncated by the frequency xlating FIR block to limit interefering signals) and the bottom being the output from the multiply block producing a CW carrier at 4x the Doppler frequency,

[My Image](https://github.com/ScottTilley/Artemis1/blob/main/RnD/OQPSK_Doppler_extract1.png)

Finally I range the flowgraph and outputed the data repeating from the file above and perform an FFT with STRF and ploted the results.  You can see the increasing Doppler from Artemis 1 just before the first TCM burn.

[My Image](https://github.com/ScottTilley/Artemis1/blob/main/RnD/OQPSK_Doppler_extract3.png)


