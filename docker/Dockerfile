FROM conda/miniconda3

RUN apt update && apt install -y git libxrender1 parallel

RUN conda install -c rdkit rdkit && conda clean -a -y

RUN pip install git+https://github.com/alperyilmaz/mol2vec

WORKDIR /data
