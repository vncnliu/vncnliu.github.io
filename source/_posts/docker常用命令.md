docker cp F:/download/jdk-8u51-linux-x64.gz 745b66cd78de:/tmp
$ sudo docker run -d -p 5000:5000 training/webapp python app.py