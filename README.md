# BeatGAN - Generating Drum Loops via GANs

## tl;dr
See the **generated_outputs** folder for samples created by the model and the **similarity_metric_examples** folder for examples of training data. I'll inline this in a future update as it looks like GitHub doesn't let you play audio directly in the browser.  

## Abstract
GANs have been used extensively for [Image Synthesis](https://arxiv.org/abs/1711.11585), [Image to Image Translation](https://arxiv.org/abs/1711.09020), and many other image tasks. More recently, they have been applied to the task of [raw audio synthesis](https://arxiv.org/pdf/1802.04208.pdf) by Donahue et al with their WaveGAN architecture. Previous audio generation techniques relied on HMMs, autoregressive models, or applying image-based techniques to spectrograms (images of waveforms in the time-domain). Donahue et al demonstrated that by applying a 1D version of DCGAN directly to regular (normalized) audio files, one could generate high quality samples of human speech superior to these previously discovered techniques. The papers's goal was to generate speech, but the authors also applied their GAN to a small dataset of drum hits and were able to produce high quality samples. This project aims to explore the ability of the same model architecture to generate significantly more complex audio patterns, namely drum beats. In particular, using the architecture presented in the paper, a GAN is trained to generate the first bar in a 4 bar drum pattern. The model is able to produce high quality samples close to par with those published in the original paper. Furthermore, the majority of the GAN's outputs are new beats, rather than  re-hashes of the training data, as measured by a quantitative similarity metric and user testing. 

## Generating Loops
WaveGAN's generator outputs vectors of shape _(16384, c)_, where _c_ is the number of audio channels. This allows the number of params in WaveGAN to be the same as in DCGAN. A larger model would require more parameters, thus [significantly more data](https://arxiv.org/pdf/1711.06491.pdf). 

44.1 khz is the standard sample rate for nearly all professional digital audio. However, at 44.1khz, the generator can only output .37 seconds of audio (16384/44100). This is far too short to contain meaningful audio. I chose to sample the audio files at 14.7 khz instead of 44.1khz. This allowed me to generate 1.11 seconds of audio with the generator and also made for a very easy resampling process - all I had to do was take every third sample in the original stream.  

1.11 seconds is still too small for an entire four bar pattern. However, the vast majority of beat patterns follow an AAAA, AAAB, AABB, or ABAB structure, meaning that the first bar of the pattern gives you at least half of the entire pattern. At 120-125 bpm, the first bar of a 4/4 drum pattern is just under one second long. Thus, a valid objective for the model is to generate the first bar in a 4/4 four bar pattern at 120-125bpm using a sample rate of 14.7khz. This lets the model generate single bars which can be stacked to form loops. Furthermore, forcing the model to use a consistent bpm reduces the amount of training data needed. 

## The Training Data
I trained the model using the [ULTIMATE DRUM LOOPS](splice.com/sounds/toolroom-records/ultimate-drum-loops) sample pack of stereo modern, EDM-style drum loops. There are 434 samples in total, all of which are either 124 of 125 bpm (the ideal tempo). Furthermore, the production quality appears to be constant across all samples, meaning variability in production technique, preset quality etc is not present. 

The samples have a native sample rate of 44.1khz so they are downsampled to 14.7khz by just taking every third value. The samples are all labeled with their bpm (124 or 125) so the first bar is just the first 14112 (or 14226) values in the sample. I throw away all audio values after the first bar, truncating the sampels down to a length of 14112 (14226). 

Since we are only using the first bar, the 434 samples equate to just 7 minutes of training audio. The datasets used by Donahue et al were all over 20 minutes long, so more data was neeeded. To augment the ULTIMATE DRUM LOOPS data, I wrote a simple white noise function and applied it five times to each original sample. This gave me a training data set of 2170 samples, each 1 second long (35 minutes of audio in total). 

## Training the Model
BeatGAN uses the same architecture and hyperparameters as WaveGAN (both described in the appendix of the WaveGAN paper). The one modification is that I use 2 channels (_c=2_) instead of one because the training data is stereo. 

I trained the model for 6000 epochs (200k iterations) shuffling the training data before every epoch. I stored the final weights for G and D in the **weights** folder and used these weights for all model evaluation tasks. The model took about 24 hours to train on a Tesla P100 GPU in GCP (CPU was an 8-core intel with 16GB of RAM). 

## Output Evaluation

### Quality
Evaluating quality can be difficult as it is often subjective for generative models. Donahue et al relied on a user study. I didn't have access to a similar resource, so I just collected feedback from a number of friends, some of whom are in the music industry. The general consensus was that the outputs sound like legitmate beats. There weren't any weird artifacts or odd sounds that 'gave away' the fact that this was generated, not producer, music. The listeners would've liked higher audio quality, but this seems to be an issue with the sample rate rather than the model, as most listeners agreed the outputs were of similar quality to the downsampled training data. Most listeners also wanted to see more diversity in the model output. This can likely be fixed with more and more diverse training data. 

### Novelty
One potential issue when training GANs is the GAN collapsing and simply outputing training data instead of new samples. To ensure BeatGAN's outputs were actually unique, I created a similarity score to measure the uniqueness of each generated batch (function _compute_similarity_score_ in the notebook) 

For two 1D audio files _A,B_ of the same length, I defined _sum((A-B)^2))/sum(A^2)_ to be the similarity of the files (lower score means more similar). I found qualitatively that the scores aligned quite well with simlarity and that a score of 0.1 or lower generally meant two files were similar. Other listeners agreed after we tested the metric on multiple generated outputs vs the training data. 

To compute the similarity score for a generated batch, I count the number of generated samples for which there is a sample in the training set that has a similarity score of 0.1 or lower. On average, only 30% to 40% of the generated outputs are similar to the training examples according to the metric, meaning the majority of beats generated by the model are actually novel. Checking the actual score values for a generated sample versus the training data confirms this. The folder **similarity_metric_examples** at the top level provides some examples of generated outputs evaluated against the input data (each example has a README). As shown in the examples, outputs with no scores <0.1 in the training data appear to be unique. More examples will be added over time. 

## Further Work
The largest extension to this work would be finding a way to produce meaningfully long samples at a high quality sample rate (e.g. 44.1khz). This would allow for more diversity in output and easier qualititative evaluation of the output, as it could then be compared against professional digital audio. 

The second largest extension to this work would be in finding ways to speed up and slow down audio tracks outside the 120-125 range without losing quality or features. This would allow for singificantly more training data to be fed into the model. As a side note, most DAWs have this scaling feature, but I have been told the quality varies a lot between DAWs (Audacity seems to have the best scaler).  

## Running the Model Yourself
Setup the environment using the environment-gpu.yml YAML. Put the training data into the **ULTIMATE_DRUM_LOOPS** folder (instructions provided in that folder's README) and run the first six cells in beat_gan.ipynb, that is, every cell up to and including the one that calls generate_batch. The generator will output 40 samples into the **generated_output** directory and compute the similarity metric for the generated samples. 


