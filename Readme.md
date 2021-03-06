# SingleVC

SingleVC performs any-to-one VC, which is an important component of [MediumVC project](https://github.com/BrightGu/MediumVC). 
Here is the official implementation of the paper, [MediumVC](http://arxiv.org/abs/2110.02500).

The following are the overall model architecture.
![Model architecture](any2one/demo_page/image/any2one.png)

For the audio samples, please refer to our [demo page](https://brightgu.github.io/SingleVC/). The more details can be found in "any2one/demo_page/ConvertedSpeeches/".

### Envs
You can install the dependencies with
```bash
pip install -r requirements.txt
```

### PSDR
[PSDR](http://www.guitarpitchshifter.com/algorithm.html) means scaling F0 and correlative harmonics with duration remained, which intuitively modifying the speaker-related information while maintaining linguistic content and prosodic information. PSDR can be used as a data augment strategy for VC by producing fake parallel corpus. To verify its feasibility that slight pitch shifts don't affect content information,  we measure the word error rate(WER) between source speeches and pitch-shifted speeches through [Wav2Vec2-based ASR System](https://github.com/huggingface/transformers). The speeches of p249(female) from VCTK Corupsis selected, and [pyrubberband](https://github.com/bmcfee/pyrubberband) is utilized to  execute PSDR. Table indicates that when S in -6~4, the strategy applies to VC with acceptable WERs.

| S | -7 | -6 | -5 |0 | 3| 4 | 5 |
| :------:| :------: | :------: |:------: |:------: |:------: |:------: |:------: |
| WER(%) | 40.51 | 25.79 |17.25 |0 |17.27 |25.21 |48.14 |


### Vocoder
The [HiFi-GAN](https://github.com/jik876/hifi-gan) vocoder is employed to convert log mel-spectrograms to waveforms. The model is trained on universal datasets with 13.93M parameters. Through our evaluation, it can synthesize 22.05 kHz high-fidelity speeches over 4.0 MOS, even in cross-language or noisy environments.

## pretrained models
You can download the pretrained model as well as the vocoder following the [link](https://drive.google.com/file/d/1yV9cCne7piqBI9vng13JDdLuRlMkTbZR/view?usp=sharing), and then edit the config file any2one/infer/infer_config.yaml.  Infer corpus should be organized as test22050/*.wav
You can convert an list of  utterances, e.g.
```bash
python any2one/infer/infer.py
```
## Train from scratch

### select acceptable pitch shifts
If you want to reconstruct someone's voice, you need to calculate the acceptable pitch shifts of that person first. Edit the "any2one/tools/wav2vec_asr.py" and config the "wave_dir" as "speech16000_dir". The ASR model provided in "any2one/tools/wav2vec_asr.py" only supports English speech recognition currently. You can replace it for other languages. In our test, the acceptable pitch shifts of  p249 in VCTK-Corups are [-6,4].
```bash
python any2one/tools.wav2vec_asr.py
```
**tips:** In practice, it performers a higher probability of success to build  female voices than male voices . Compared to males, the periodic patterns of females perform more stable due to the higher frequency resolution. 

The train corpus should be organized as vctk22050/p249/*.wav
```bash
python any2one/solver.py
```
### Preprocessing
1. The model is trained with random pitch shifted speeches processed in real-time. If you want to speed up the training, please refer the code in "any2one/meldataset.py" to have data preprocessed.
2. If use preprocess method in HiFi-GAN vocoder, the training will take about one day with TITAN Xp, and the performances will be more robust. However, using preprocess method in WaveRNN,  the training will just spend three hours.

