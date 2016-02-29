# yelp-challenge-2016-DeepLearning
This repository is the code for training a word level CNN for sentiment classification on [Yelp Challenge 2016 dataset](https://www.yelp.com/dataset_challenge). The code deals with the `yelp_academic_dataset_review.json` sub dataset, which has two columns namely "stars" and "text", and it contains about 5 million rows. The "stars" is the customers' rating from 1 to 5, the "text" field is the customers' review sentence. 

In order to train the model on my local machine in reasonal time, I subsampled 1 million from it and get 399850 rows after removing rows with `null` values. The model achieved 77.91% accuracy on the validation set after 2 epoch, which you can chech from the `train_keras_embedding.ipynb` file.

## Components
This repository contains the following components:
* `json-csv.py` : This is the data preprocessing file, it converts the `yelp_academic_dataset_review.json` file to a `review.csv` in the same directory. Notice that the `json` file is actually not a valid json file that you can load directly, but each its row is. So you have to process it line by line.
* `Word2VecUtility.py` : This file is borrowed from Kaggle's [word2vec tutorial](https://github.com/wendykan/DeepLearningMovies). It processes each sentence into word list or sentence list.
* `word2vec_model.ipynb` : This is used to train a Google's `word2vec` model on all the 1859888 sentences(processed from all the 399850 reviews using `Word2VecUtility.py`. Each word would be represented by a 300 dimensional vector. The final model named `300features_40minwords_10context` is written to disk.
* `train_with_word2vec_embedding.ipynb`: After we trained a `word2vec` model, we can transform each review into a fixed length of words with each word represented by its word2vec vector. In order to train the model on my machine, I set `max_length`(max number of words for each review, those with less than 50 word are padded with 0) as 50, `max_word`(max number of word features in the word2vec model) as 5000. To do this, I first built a word index dictionary to represent each review as a list of word indexes. The indexes of word in word2vec model are all increased by 3. I let all reviews begin with the index 1 and all the words whose indexes are outside of [3 and 5003] be replaced by 2. After this, for each review, I map its words to the corresponding word2vec vectors, which mean each review is now a (50, 300) matrix. Then I trained a 1D Convelutional Neural Net model on this newly embedded data set (the shape of input data is (399850, 50, 300)). Unfortunately, my machine was not able to train it due to the memory size. So I turn to replace word2vec embedding as Keras' built-in embedding layer.
* `train_keras_embedding.ipynb`: This model is similar to the previous one. The only difference is on the embedding layer. To use keras embedding layer, I feed in the review data as its word indexes representation(list of list of word indexed) as we did in `train_with_word2vec_embedding.ipynb`. I set the `embedding_dims` = 100. To be more specific, the shape of input data is (399850, 50), the output of embedding lay has a shape of (399850, 50, 100). The architecture of this model is : Embedding layer - Dropout - Convolution1D - MaxPooling1D - Full Connected layer - Dropout - Relu activation - Sigmoid (with binary cross entropy loss). Other model parameters are to be seen in this file. It is trained on 319880 samples, validated on 79970 samples, after 2 epoch, I got train acc: 0.7791 and val_acc: 0.7761.
