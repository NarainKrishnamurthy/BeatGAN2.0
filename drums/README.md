# Original WaveGAN Drum Hit Dataset

This folder is where I stored the drum hit data from the original WaveGAN paper. This is the dataset: http://www.hexawe.net/mess/200.Drum.Machines/drums.zip. To use it with my model:

1) Unzip the dataset
2) Store in this drums/ folder. Your folder structure should be: 

* drums/Acetone Rhythm Ace
* drums/Acetone Rhythm King
* drums/Acetone Rhythm Master

etc. 
3) Compute X_train by running the cell with the function load_wavegan_paper_drumhit_data. Then run the cell with the train function. 
