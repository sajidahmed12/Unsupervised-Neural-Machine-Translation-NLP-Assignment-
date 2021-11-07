# Neural Machine Translation (NMT) From Bangla To English

Neural Machine Translation (NMT) is one of the most common type of machine translation method that relies upon neural network models to build statistical models with the end goal of translation. The essential advantage of NMT is that it gives a solitary system that can be prepared to unravel the source and target text.One of the most well recognized NMT library is Multilingual Unsupervised and Supervised Embeddings ([MUSE](https://github.com/facebookresearch/MUSE)).This python library is  for multilingual word embeddings, whose goal is to provide the community with:

* state-of-the-art multilingual word embeddings ([fastText](https://github.com/facebookresearch/fastText/tree/main/alignment) embeddings aligned in a common space)
* large-scale high-quality bilingual dictionaries for training and evaluation

This approach includes both *supervised* method that uses a bilingual dictionary or identical character strings, and also *unsupervised* method that does not use any parallel data (see [Word Translation without Parallel Data](https://arxiv.org/pdf/1710.04087.pdf) for more details).

## Dependencies

* Python 2/3 with [NumPy](http://www.numpy.org/)/[SciPy](https://www.scipy.org/)
* [PyTorch](http://pytorch.org/)
* [Faiss](https://github.com/facebookresearch/faiss) (recommended) for fast nearest neighbor search (CPU or GPU).

## Datasets

To download monolingual and cross-lingual word embeddings evaluation datasets:
* Our 110 [bilingual dictionaries](https://github.com/facebookresearch/MUSE#ground-truth-bilingual-dictionaries)
* 28 monolingual word similarity tasks for 6 languages, and the English word analogy task
* Cross-lingual word similarity tasks from [SemEval2017](http://alt.qcri.org/semeval2017/task2/)
* Sentence translation retrieval with [Europarl](http://www.statmt.org/europarl/) corpora

You can simply run:

```bash
cd data/
wget https://dl.fbaipublicfiles.com/arrival/vectors.tar.gz
wget https://dl.fbaipublicfiles.com/arrival/wordsim.tar.gz
wget https://dl.fbaipublicfiles.com/arrival/dictionaries.tar.gz
```

Alternatively, you can also download the data with:

```bash
cd data/
./get_evaluation.sh
```

*Note: Requires bash 4. The download of Europarl is disabled by default (slow), you can enable it [here](https://github.com/facebookresearch/MUSE/blob/master/data/get_evaluation.sh#L99-L100).*

## Methodology 

The key idea is to create a common latent space between the two languages (or domains),in our case English and Bengali, and learn to translate by reconstructing in both domains

Principles:

* The model has to be able to reconstruct a sentence in a given language from a noisy version of it, as in standard denoising auto-encoders
* The model also learns to reconstruct any source sentence given a noisy translation of the same sentence in the target domain,and vice versa.

![Big Picture](./docs/Big_Picture.PNG)

Unsupervised Encoding-Decoding

An encoder and a decoder, respectively responsible for encoding source and target sentences to a latent space, and to decode from that latent space to the source or the target domain.A single encoder and a single decoder for used in both the domains. The only difference when applying these modules to different languages is the choice of lookup tables.

![Auto-Encoding and Translation Model Architecture](./docs/Model_Arch_Auto_Encoding.PNG)

Our Performed steps: 

* Encode-Decode using generic data which was parallelized (Provided by MUSE)  
* Performed Seq-to-Seq Enc-Dec-LSTM training with that Parallel Dataset. The results were noteworthy with the provided dataset but requires more testing with other datasets.
* Then changed to FastText Alligned Vector and generated the Word Embedding From Vector using MUSE

### Monolingual word embeddings
For pre-trained monolingual word embeddings, we highly recommend [fastText Wikipedia embeddings](https://fasttext.cc/docs/en/pretrained-vectors.html), or using [fastText](https://github.com/facebookresearch/fastText) to train your own word embeddings from your corpus.

You can download the English (en) embeddings this way:
```bash
# English fastText Wikipedia embeddings
curl -Lo data/wiki.en.vec https://dl.fbaipublicfiles.com/fasttext/vectors-wiki/wiki.en.vec
```
### Align monolingual word embeddings
(https://en.wikipedia.org/wiki/Orthogonal_Procrustes_problem) alignment.
* **Unsupervised**: without any parallel data or anchor point, learn a mapping from the source to the target space using adversarial training and (iterative) Procrustes refinement.

#### The unsupervised way: adversarial training and refinement (CPU|GPU)
To learn a mapping using adversarial training and iterative Procrustes refinement, run:
```bash
python unsupervised.py --src_lang en --tgt_lang es --src_emb data/wiki.en.vec --tgt_emb data/wiki.es.vec --n_refinement 5
```
By default, the validation metric is the mean cosine of word pairs from a synthetic dictionary built with CSLS (Cross-domain similarity local scaling). For some language pairs (e.g. En-Zh),
we recommend to center the embeddings using `--normalize_embeddings center`.

#### Evaluate monolingual or cross-lingual embeddings (CPU|GPU)

We also include a simple script to evaluate the quality of monolingual or cross-lingual word embeddings on several tasks:

**Monolingual**
```bash
python evaluate.py --src_lang en --src_emb data/wiki.en.vec --max_vocab 200000
```

**Cross-lingual**
```bash
python evaluate.py --src_lang en --tgt_lang es --src_emb data/wiki.en-es.en.vec --tgt_emb data/wiki.en-es.es.vec --max_vocab 200000
```
### Word embedding format
By default, the aligned embeddings are exported to a text format at the end of experiments: `--export txt`. Exporting embeddings to a text file can take a while if you have a lot of embeddings. For a very fast export, you can set `--export pth` to export the embeddings in a PyTorch binary file, or simply disable the export (`--export ""`).

When loading embeddings, the model can load:
* PyTorch binary files previously generated by MUSE (.pth files)
* fastText binary files previously generated by fastText (.bin files)
* text files (text file with one word embedding per line)

The two first options are very fast and can load 1 million embeddings in a few seconds, while loading text files can take a while.


#### Multilingual word Embeddings
We release fastText Wikipedia **supervised** word embeddings for **30** languages, aligned in a **single vector space**.

| | | | | | |
|---|---|---|---|---|---|
| Bengali: [full](https://dl.fbaipublicfiles.com/arrival/dictionaries/bn-en.txt) [train](https://dl.fbaipublicfiles.com/arrival/dictionaries/bn-en.0-5000.txt) [test](https://dl.fbaipublicfiles.com/arrival/dictionaries/bn-en.5000-6500.txt) 

We provide multilingual embeddings and ground-truth bilingual dictionaries. These embeddings are fastText embeddings that have been aligned in a common space.

## Results

### Visualization of the multilingual word embedding space

**Nearest Neighbors(NN)**

Precision Obtained at

k = 1: 32.333333 <br />
k = 5: 52.666667 <br />
k = 10: 60.266667 <br />

**K nearest neighbors(KNN)** for **Cross-domain similarity local scaling (CSLS)**  

Precision Obtained at 

k = 1: 35.933333 <br />
k = 5: 56.200000 <br />
k = 10: 63.200000 <br />

![Similar word embeddings in same latent space for Bengali and English words](./docs/Word_Embeddings.PNG)

Text Pre-processing output
<!-- To Resize need to use http link
![Text Preprocessing](link_here =250x250) 
-->
![Text Preprocessing](./docs/text_preprocessing.jpg)


NMT Sample Output
<!-- To Resize need to use http link
![NMT Output](link_here =250x250) 
-->
![NMT Output](./docs/NMT_Output.png)





###  Author Names and Contribution

Suprervisor
* Dr. Nabeel Mohammed

Authors

* Zahin Akram- 1610618042
* Sajid Ahmed- 1610364042
* Arifuzzaman Arman- 1610551042
* Md Rakib Imtiaz- 1610294642

Contact: [sajid.ahmed1@northsouth.edu](mailto:[sajid.ahmed1@northsouth.edu)

* All 4 members of the group worked together in understanding the paper 
through by separate and then reading the paper together. All members together began
the training of word embedding through MUSE. It was Sajid and Zahin who looked at the 
code deeply and worked in editing the necessary portion of what was required. Sajid
later ran the model and generated the mapping that was later shown to Sir.

* Sajid looked at the models that were to be used for training once the first part was over.
The decision of the model was decided by Zahin and Sajid together as they were primarily in charge
of the assignment. Once the decision of model was made, further adjustments were made together.
Sajid ran the models necessary in his PC. The dataset was also provided by Sajid.
Sajid oversaw the training in his PC. Zahin and Sajid made final adjustments.

#### Notes:

* A lot of issue was the specifications of available systems. The embeddings for the English
vocabulary was extremely large and had to be redone using fewer words. We wrote some code for 
specific amount of vocabulary however even that was not working. We tested that code on some other
files and they worked fine. The problem was that the file was too large that it couldn't be opened.
We had to edit the original final to address the memory error.

* The words chosen were based on frequency. However, most frequent English and Bangla words are different. There was also the Unicode error that took a while to solve. It was later solved by the Temporary use of an UNIX system but it had low specifications and wasn't available for extensive use. We had a lot of problems in training and evaluating training as each training took very long and not the same vector file could be used for every model. We also had infrequent type error issues
as well as environmental issues when running multiple models.

* The work would have been better if we had unrestrained access to a powerful system for an extensive period of 2 weeks so we could test all our models with varying combinations.

## Important Links

#### Download the trained_model.pt file from the following link: 

https://drive.google.com/file/d/17-RFIitkcnphSQZ_CkxN_203laTO8lVH/view?usp=sharing

#### -- English To Bangla Translation --

* python3 translate.py -s data/eng_test.txt -sl e -t out_ben.txt

#### -- Bangla To English Translation --

* python3 translate.py -s data/ben_test.txt -sl b -t out_eng.txt

also can be run through demo.ipynb Directly From Google.colab

#### Download the Vectors from these Fast Text links:

Word vectors for 157 languages (Fasttext) - https://fasttext.cc/docs/en/crawl-vectors.html

vectors-bn.txt -- https://dl.fbaipublicfiles.com/fasttext/vectors-crawl/cc.bn.300.vec.gz

vectors-en.txt -- https://dl.fbaipublicfiles.com/fasttext/vectors-crawl/cc.en.300.vec.gz
and place then in ./data/vec/vectors-{bn.en}.txt

## References

* [T. Mikolov, Q. V Le, I. Sutskever - Exploiting similarities among languages for machine translation, 2013](https://arxiv.org/abs/1309.4168)
* [G. Dinu, A. Lazaridou, M. Baroni - Improving zero-shot learning by mitigating the hubness problem, 2015](https://arxiv.org/abs/1412.6568)
* [S. L Smith, D. HP Turban, S. Hamblin, N. Y Hammerla - Offline bilingual word vectors, orthogonal transformations and the inverted softmax, 2017](https://arxiv.org/abs/1702.03859)
* [M. Artetxe, G. Labaka, E. Agirre - Learning bilingual word embeddings with (almost) no bilingual data, 2017](https://aclanthology.coli.uni-saarland.de/papers/P17-1042/p17-1042)
* [M. Zhang, Y. Liu, H. Luan, and M. Sun - Adversarial training for unsupervised bilingual lexicon induction, 2017](https://aclanthology.coli.uni-saarland.de/papers/P17-1179/p17-1179)
* [Y. Hoshen, L. Wolf - An Iterative Closest Point Method for Unsupervised Word Translation, 2018](https://arxiv.org/abs/1801.06126)
* [A. Joulin, P. Bojanowski, T. Mikolov, E. Grave - Improving supervised bilingual mapping of word embeddings, 2018](https://arxiv.org/abs/1804.07745)
* [E. Grave, A. Joulin, Q. Berthet - Unsupervised Alignment of Embeddings with Wasserstein Procrustes, 2018](https://arxiv.org/abs/1805.11222)