## BeatGAN Training Set

The folder where I store the training data for BeatGAN, the beat generation model.  The data can be found here: https://splice.com/sounds/toolroom-records/ultimate-drum-loops. To use it with my model:

1) Download the dataset to your computer (costs like $10).
2) Copy the folder created by Splice here. Folder structure should be 

* ULTIMATE_DRUM_LOOPS/tr07_drlp_124_Believe_Full.wav
* ULTIMATE_DRUM_LOOPS/tr07_drlp_124_BigDipper_Full.wav
* ULTIMATE_DRUM_LOOPS/tr07_drlp_124_Cave_Full.wav

etc.

Running the cell with the train(6100, hp.b) call will run it with this dataset. Note, make sure you download the wavfile24.py module in the top level directory of the project as all of these wavfiles are 24 bit and wavfile24 is needed to read them. 
