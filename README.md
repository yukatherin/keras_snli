# Keras SNLI baseline example

This repository contains a simple Keras baseline to train a variety of neural networks to tackle the [Stanford Natural Language Inference (SNLI) corpus](http://nlp.stanford.edu/projects/snli/).

The aim is to determine whether a premise sentence is entailed, neutral, or contradicts a hypothesis sentence - i.e. "A soccer game with multiple males playing" entails "Some men are playing a sport" while "A black race car starts up in front of a crowd of people" contradicts "A man is driving down a lonely road".

The model architecture is:

+ Extract a 300D word vector from the fixed GloVe vocabulary
+ Pass the 300D word vector through a ReLU "translation" layer
+ Encode the premise and hypothesis sentences using the same encoder (summation, GRU, LSTM, ...)
+ Concatenate the two 300D resulting sentence embeddings
+ 3 layers of 600D ReLU layers
+ 3 way softmax

![Visual image description of the model](https://rawgit.com/Smerity/keras_snli/master/snli_model.svg)

Training uses RMSProp and stops after N epochs have passed with no improvement to the validation loss.
Following [Liu et al. 2016](http://arxiv.org/abs/1605.09090), the GloVe embeddings are not updated during training.
Following [Munkhdalai & Yu 2016](http://arxiv.org/abs/1607.04315), the out of vocabulary embeddings remain zeroed out.

One of the most important aspects when using fixed Glove embeddings with summation is the "translation" layer.
[Bowman et al. 2016](http://nlp.stanford.edu/pubs/snli_paper.pdf) use such a layer when moving from 300D to the lower dimensional 100D hidden state.
This is likely highly important for the summation method as it allows the GloVe space to be shifted before summation.
Technically when done with training the "translated" GloVe embeddings could be precomputed and this layer removed, decreasing the number of parameters, but ¯\\\_(ツ)\_/¯

The model is relatively simple yet sits at a far higher level than other comparable baselines (specifically summation, GRU, and LSTM models) listed on [the SNLI page](http://nlp.stanford.edu/projects/snli/).
The summary: don't dismiss well tuned GloVe bag of words models - they can still be competitive and are far faster to train!

Model                                              | Parameters | Train  | Validation | Test
---                                                | ---        | ---    | ---        | ---
300D sum(word vectors) + 3 x 600D ReLU (this code) | 1.2m       | 0.831  | 0.823      | 0.825
300D GRU + 3 x 600D ReLU (this code)               | 1.7m       | 0.843  | 0.830      | 0.823
300D LSTM + 3 x 600D ReLU (this code)              | 1.9m       | 0.855  | 0.829      | 0.823
300D GRU (recurrent dropout) + 3 x 600D ReLU (this code)               | 1.7m       | 0.844  | 0.832      | 0.832
300D LSTM (recurrent dropout) + 3 x 600D ReLU (this code)               | 1.9m       | 0.852  | 0.836      | 0.827
--                                                 | ---        | ---    | ---        | ---
300D LSTM encoders (Bowman et al. 2016)            | 3.0m       | 0.839  | -          | 0.806
1024D GRU w/ unsupervised 'skip-thoughts' pre-training (Vendrov et al. 2015) | 15m | 0.988 | - | 0.814
300D Tree-based CNN encoders (Mou et al. 2015)     | 3.5m       | 0.833  | -          | 0.821
300D SPINN-PI encoders (Bowman et al. 2016)        | 3.7m       | 0.892  | -          | 0.832
600D (300+300) BiLSTM encoders (Liu et al. 2016)   | 3.5m       | 0.833  | -          | 0.834

Only the numbers for pure sentential embedding models are shown here.
The [SNLI homepage](http://nlp.stanford.edu/projects/snli/) shows the full list of models where attentional models perform better.
If I've missed including any comparable models, submit a pull request.

All models could benefit from a more thorough evaluation and/or grid search as the existing parameters are guesstimates inspired by various papers (Bowman et al. 2015, Bowman et al. 2016, Liu et al. 2016).
Only when the GRUs and LSTMs feature recurrent dropout (`dropout_U`) do they consistently beat the summation of word embeddings.
Further work should be done exploring the hyperparameters of the GRU and LSTM.
