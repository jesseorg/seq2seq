# seq2seq
Fork of Google's seq2seq 202010221606BJ


  A fork of google's seq2seq 202010221606BJ
  This came from
https://google.github.io/seq2seq/getting_started/

after fixing some errors via:

	https://medium.com/@aloofness54/fix-google-seq2seq-installation-errors-4a1155b13f73

#seq2seq is base on the structure given by Cho et al.
#https://arxiv.org/pdf/1406.1078.pdf
#here is some context:
#http://web.stanford.edu/class/cs20si/lectures/slides_13.pdf



# Howto

make an example text:

	./makeExampleText.sh 

or

	head  ../seq2seq/nmt_data/birdsOfEmpire/train/sources.txt  > example_text.txt

This will copy some text from the training content so that 
we can test the translation.
note the newly created file is named 'example_text.txt'.

run 'python postText.py'

	python postText.py

this will read in 'example_text.txt', convert it to a formatted JSON doc 
called 'example_text.json'


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
	stem:	00001_0 


The server will now communicate with the dp tensorflow server {server}.
(wait a minute, the server ("dp") is on a crontab)


dp server runs:

	wget http://{server}/s2sdoc/00001_0.json -O trans.json

This gets the current subitted translation job(s).

Then runs:

	python3  ${HOME}/s2s_project/seq2seq/parseTransJSON.py 

This converts the JSON 'trans.json' into files that seq2seq can read, 
'trans.txt'. The server then proceeds to run seq2seq with the 
default model checkpoint, on 'trans.txt'.



