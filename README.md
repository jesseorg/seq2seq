# seq2seq


Seq2Seq came from
	https://google.github.io/seq2seq/getting_started/

after fixing some errors via:

	https://medium.com/@aloofness54/fix-google-seq2seq-installation-errors-4a1155b13f73

#seq2seq is base on the structure given by Cho et al.
#https://arxiv.org/pdf/1406.1078.pdf
#here is some context:
#http://web.stanford.edu/class/cs20si/lectures/slides_13.pdf





# Howto

First ssh in, then enter the Application Code directory:
	
	cd /org/dp/s2s_project/seq2seq

We will need to load a python virtual environment, which gives us access to all the libraries that we need, this is accessed by 'workon dfl':

	workon dfl


make an example text:

	./makeExampleText.sh

or

	head  ../seq2seq/nmt_data/birdsOfEmpire/train/sources.txt  > example_text.txt

This will copy some text from the training content so that we can test the translation.
Note the newly created file is named 'example_text.txt'.
You can view this text by typing:

	more example_text.txt


We will wrap this text in JSON and post it to the public server by running 'python postText.py':

	python postText.py

This will read in 'example_text.txt', convert it to a formatted JSON doc called 'example_text.json'

It will then send the doc to:

	http://{server}/cgi-bin/filer.py

as an HTTP POST in the following data-structure -

	data:
	{"nodes":[{"x":127.87355321789117,"y":39.306591596670735,"text":"hello from web, this is any text","isAcceptState":false}],"links":[]}

	fileURL:
	/s2sdoc

	stem:
	00001_0


For example, the following URL will do the same thing,
and will store some JSON text on the server.

	http://{server}/cgi-bin/filer.py?fileURL=/s2sdoc/&stem=00001_0&data={%22nodes%22:[{%22x%22:62,%22y%22:40,%22text%22:%22certainly%20not%22,%22isAcceptState%22:false}],%22links%22:[{%22type%22:%22StartLink%22,%22node%22:5,%22text%22:%22%22,%22deltaX%22:-96.76006832873145,%22deltaY%22:-160.027070294523}]}



The server will save the file to the server at the URL:

	http://{server}/s2sdoc/00001_0.json

Because we have specified-

	fileURL: /s2sdoc
	stem: 00001_0



Then run
	./processPublicText.sh

This will translate the text and post the results to the public server.
First is gets the source text:
	wget http://{server}/s2sdoc/00001_0.json -O trans.json

This gets the current submitted translation job(s).
Then runs:

	python3  ${HOME}/s2s_project/seq2seq/parseTransJSON.py

This converts the JSON 'trans.json' into files that seq2seq can read,
'trans.txt'. The server then proceeds to run seq2seq with the
default model checkpoint, on 'trans.txt'.

Finally it runs:

	./postPredictions.py

Which sends the results back to the server for collection at:

	http://{server}/s2sdoc/00001_1.json

What we have just done is translated the text:

	http://{server}/s2sdoc/00001_0.json
	
into the text:

	http://{server}/s2sdoc/00001_1.json

Note there are some symbols in the translated text 'UNK', this means unknown word.
We can change the source text if we like, by editing the file example_text.txt.
For example,

	vim example_text.txt

and edit, save.

After editing, again run,

	python postText.py

This posts the text, then run,

	./processPublicText.sh

We can see the changes in the translation. If you are using a web browser to view the translations (as I do) make sure that the browser is not caching the file, I add some some numbers at the end of the URL to make sure the request is unique each time like:

	http://{server}//s2sdoc/00001_1.json?unique123


There are two other functions that are necessary to operate seq2seq: changing traingin data, and training models.

The current models are in:

	/org/dp/s2s_project/s2s_models/birdsOfEmpire/

These models generate bird language.

There is not a script yet to generate new model directories, so for ease of use, we can continue to use this model directory and train new models with new text. To accomplish that, if needed, just copy /org/dp/s2s_project/s2s_models/birdsOfEmpire/ to a backup and delete all the models in /org/dp/s2s_project/s2s_models/birdsOfEmpire/ then train on new data.

The training data is in:

	/org/dp/s2s_project/seq2seq/nmt_data/birdsOfEmpire/

Inside of which are 'dev' and 'train' directories.

The 'train' directory contains the data we train our model on. The files that we use are:

	-rw-r--r-- 1 cyrus cyrus 1.2M Nov 13 02:08 sources.txt
	-rw-r--r-- 1 cyrus cyrus 705K Nov 13 02:08 targets.txt
	-rw-r--r-- 1 cyrus cyrus  99K Nov 13 00:18 vocab.sources.txt
	-rw-r--r-- 1 cyrus cyrus 102K Nov 12 22:50 vocab.targets.txt

 The files that we use are formatted and matched line for line.

sources.txt and targets.txt are line for line pairs of text, source and translation. These are pairs that our model should learn.

vocab.sources.txt and vocab.targets.txt are vocabulary lists of all the words in the respective texts. These lists must be paired word equivalents of equal length and formatted as:

	Word[TAB]Number

	He 0
	knew 1
	that 2
	as 3
	one 4
	of 5
	the 6
	founders 7
	Canada 8

Words can have more than one corresponding equivalent, and in this way complex contextual meaning arrises.

There are other files in that directory, which are longer text files and originals, they are not needed by seq2seq training.

	/org/dp/s2s_project/seq2seq/nmt_data/birdsOfEmpire/train$ ls -laht
	total 26M
	-rw-r--r-- 1 cyrus cyrus 8.9M Nov 12 21:23 put-all_birds.txt
	-rw-r--r-- 1 cyrus cyrus  15M Nov 12 21:19 put-empire.txt
	-rw-r--r-- 1 cyrus cyrus  99K Nov 12 21:12 put-all_words_empire_ascii.txt
	-rw-r--r-- 1 cyrus cyrus 102K Nov 12 21:12 put-all_words_birds_ascii.txt


The contents of the 'dev' directory,

	/org/dp/s2s_project/seq2seq/nmt_data/birdsOfEmpire/dev

This is for simultaneous current training texts that we wish to also train on. Our setup is simple and although we keep this function for multiple texts they are not used in the training, I have just copied the current training texts into the 'dev' training directory.

The training script is called 'train.sh', this will run training from the currently set up dataset and directory structure.

When it is training the output will look like -

	INFO:tensorflow:global_step/sec: 13.5589
	I0105 21:47:33.293308 139995953059648 basic_session_run_hooks.py:692] global_step/sec: 13.5589
	INFO:tensorflow:loss = 1.3625761, step = 107700 (7.375 sec)
	I0105 21:47:33.294406 139995953059648 basic_session_run_hooks.py:260] loss = 1.3625761, step = 107700 (7.375 sec)


You can stop the training with control-c, it is running in the foreground.

There are some warnings and errors, but they are not critical. For example -

	>>2021-01-05 21:47:57.979252: W tensorflow/core/grappler/utils/graph_view.cc:830] No registered 'att_sum_dot' OpKernel for GPU devices compatible with node {{node model/att_seq2seq/decode/attention_decoder/decoder/while/attention/att_sum_dot}}.  Registered:  <no registered kernels>

This doesn't affect training.


The training text can be replaced with any 8-byte data and the system will still operate the same. The architecture is symbolic though, using a vocabulary list (which doesn't have to be text), so giving it waveforms would result is a very choppy signal. But giving it FFT data or midi seems like it would work.



//202101061254jesse


Last a note about changing the paths -
### NOTE vocab path is hard coded in -

	/org/dp/s2s_project/s2s_models/birdsOfEmpire/train_options.json

- look for -

	"vocab_source": "/org/dp/s2s_project/seq2seq/nmt_data/birdsOfEmpire/train/vocab.sources.txt", "vocab_target":"/org		/dp/s2s_project/seq2seq/nmt_data/birdsOfEmpire/train/vocab.targets.txt"}

Don't know how to remove this at the moment - it is a structure in the original seq2seq code.
###
