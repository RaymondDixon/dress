# Sentence Simplification with Deep Reinforcement Learning
This is an implmentation of the DRESS (**D**eep **RE**inforcement **S**entence **S**implification) model described in [Sentence Simplification with Deep Reinforcement Learning](https://homepages.inf.ed.ac.uk/s1270921/res/papers/emnlp2017.pdf)


# Datasets
The *newsela*, *wikismall* and *wikilarge* datasets can be downloaded [here](https://drive.google.com/open?id=0B6-YKFW-MnbOWHlSLXRqUTZlWms)

Copyright of the *newsela* dataset belongs to https://newsela.com.
Therefore, the *newsela* dataset is anonymized (words are replaced with random numbers). If you have any question about the datasets, feel free to contact us.

# Dependencies
* [CUDA 7.5](http://www.nvidia.com/object/cuda_home_new.html)
* [Torch](https://github.com/torch)
* [fb-python](https://github.com/facebook/fblualib/tree/master/fblualib/python)
* [pysari](https://github.com/XingxingZhang/pysari)

Note that this model is tested using an old version of torch (available [here](https://drive.google.com/open?id=0B6-YKFW-MnbOZ0gxNk56MjhQWjA))


# Train a Reinforcement Learning Simplification Model

## Step 1: Train an Encoder-Decoder Attention Model
```
CUDA_VISIBLE_DEVICES=$ID th train.lua --learnZ --useGPU \
    --model EncDecAWE \
    --attention dot \
    --seqLen 85 \
    --freqCut 4 \
    --nhid 256 \
    --nin 300 \
    --nlayers 2 \
    --dropout 0.2 \
    --lr $lr \
    --valid $valid \
    --test $test \
    --optimMethod Adam \
    --save $model \
    --train $train \
    --validout $validout --testout $testout \
    --batchSize 32 \
    --validBatchSize 32 \
    --maxEpoch 30 \
    --wordEmbedding $wembed \
    --embedOption fineTune \
    --fineTuneFactor 0 \
    | tee $log
```
Details see `experiments/wikilarge/encdeca/train.sh`. Note in `newsela` and `wikismall` datasets, you should use `--freqCut 3`. 

If you want to generate simplifications from a pre-trained Encoder-Decoder Attention model, use the following command:
```
CUDA_VISIBLE_DEVICES=3 th generate_pipeline.lua \
    --modelPath $model \
    --dataPath $data.test \
    --outPathRaw $output.test \
    --oriDataPath $oridata.test \
    --oriMapPath $orimap | tee $log.test
```
Details see `experiments/wikilarge/encdeca/generate/run_std.sh`.

## Step 2: Train a Language Model for the Fluency Reward
See details in `experiments/wikilarge/dress/train_lm.sh`

## Step 3: Train a Sequence Auto-Encoder for the Relevance Reward
See details in `experiments/wikilarge/dress/train_auto_encoder.sh`

## Step 4: Train a Reinforcement Learning Model
See details in `experiments/wikilarge/dress/train_dress.sh`. Run a pre-trained `DRESS` model using this script `experiments/wikilarge/dress/generate/dress/run_std.sh`.

## Step 5: Train a Lexical Simplification Model 
To train a lexical simplification model, you need to obtain soft word alignments in the training data, which are assigned by a pre-trained Encoder-Decoder Attention model. See details in `experiments/wikilarge/dress/run_align.sh`.

After you obtain the alignments, you can train a lexical simplification model using `experiments/wikilarge/dress/train_lexical_simp.sh`.

Lastly, you can apply the lexical simplification model with DRESS `experiments/wikilarge/dress/generate/dress-ls/run_std.sh`.

# Pre-trained Models
https://drive.google.com/open?id=0B6-YKFW-MnbOTVRMSURFbXYxNjg

