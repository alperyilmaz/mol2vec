# smaller model model_300dim.pkl

Pre-trained Mol2vec model which was trained on 20 million compounds downloaded from ZINC using:

* radius 1
* UNK to replace all identifiers that appear less than 4 times
* skip-gram and window size of 10
* resulting in 300 dimensional embeddings

# larger model model_1bln.pkl

Nearly 1 billion drug-like chemicals from ZINC20 were used for training. Roughly, following steps were followed (codes can be run within container using the code: `sudo docker run -it --rm -v $(pwd):/data alperyilmaz/conda-mol2vec bash`

## Download smi file

urls for separate tranches can be downloed from [ZINC20](http://zinc20.docking.org/tranches/home) site. Some urls might fail, so let's keep list of downloaded files after download is complete.

```bash
cat zinc20_druglike.curl | bash

ls -1 ????.smi | cut -f1 -d. > tranch_list
```

## Generate corpus

Start `mol2vec corpus` in parallel for each smi file, notice that there's no min occurrence threshold, so that each file is processed in parallel. This step took 6.5 hrs at c5.18xlarge (AWS) which has 72 threads.

```bash
mkdir -p corpus
mkdir logs-corpus
parallel  --delay 0.5 -j 36 --joblog logs-corpus/runtask.log "mol2vec corpus -i {}.smi -o corpus/{}.cp -r 1 -j 2 &> logs-corpus/{}_.log" :::: tranch_list   
```

## Training

Although mol2vec allows parallel processing in training step, most of the time is spend in reading the large file which is serial. This step took 94 mins ( 157 mins real cpu time at c5.18xlarge)

```bash
mol2vec train -i <(cat corpus/*.cp) -o model_1bln.pkl -d 300 -w 10 -m skip-gram --threshold 3 -j 72
```

