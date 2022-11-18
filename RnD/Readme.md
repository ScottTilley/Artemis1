Using a suggestion from Daniel Estévez I've implemented a flow graph to study the viability of this method to extract Doppler data from Artemis 1's OQPSK modulation.  As my antenna is too small to demodulate the signal to obtain Doppler data we are trying different techniques to extract useful Doppler data from the signal.

I made a recording of Artemis 1 while near earth and it was sending idle data.  This can be found here:
https://github.com/ScottTilley/Artemis1/blob/main/RnD/OQPSK_Doppler_extract2.png

The flow graph uses a frequency xlating FIR block to decimate and potentially shift the output as needed.  Next, per Daniel's suggestion the output is raised to the 4th power with a multiply block with four inputs of the same output from the same signal.  The output is set to a FIFO for analysis with Cees Bassa's STRF.


