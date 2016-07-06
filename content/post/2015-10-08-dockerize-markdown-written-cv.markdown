---
categories: ["Linux"]
comments: true
date: 2015-10-08T10:08:02Z
title: Dockerize Markdown Written CV
url: /2015/10/08/dockerize-markdown-written-cv/
---

### Tips
Manually run the commands for generating the CVs.     

```
$ sudo apt-get install -y build-essential
$ sudo apt-get install -y pandoc
$ sudo apt-get install -y wkhtmltopdf 
$ sudo apt-get install xvfb
$  echo 'xvfb-run --server-args="-screen 0, 1024x768x24" /usr/bin/wkhtmltopdf $*' > \
/usr/bin/wkhtmltopdf.sh
$ chmod a+x /usr/bin/wkhtmltopdf
$ chmod a+x /usr/bin/wkhtmltopdf.sh 
$ ln -s /usr/bin/wkhtmltopdf.sh /usr/local/bin/wkhtmltopdf
$ apt-get install -y ttf-wqy-zenhei
$ apt-get install -y git
$ apt-get install -y rubygems-integration ruby-dev
$ apt-get install -y libimage-exiftool-perl
$ gem install compass
$ gem install susy
$ git clone https://github.com/barraq/pandoc-moderncv.git
$ cd pandoc-moderncv/
$ mkdir -p cv/images/
$ vim cv/cv.md
$ gem install gempass
$ gem install 
$ make
#####$ apt-get install python-software-properties
#####$ apt-get install software-properties-common
$ vim Makefile 
   -> Change the wkpdf to wkhtmltopdf
$ make pdf
```

### Dockerfile
Write the Dockerfile like following:    

```
# Based on Ubuntu 14.04
FROM ubuntu:trusty

# Install Packages, via apt-get. 
RUN apt-get update && apt-get install -y \
	build-essential \
	pandoc \
	wkhtmltopdf \
	xvfb \
	ttf-wqy-zenhei \
	git \
	rubygems-integration \
	ruby-dev \
	libimage-exiftool-perl \
	python-twisted

# Now Change wkhtmltopdf
RUN echo 'xvfb-run --server-args="-screen 0, 1024x768x24" /usr/bin/wkhtmltopdf $*' > /usr/bin/wkhtmltopdf.sh
RUN chmod a+x /usr/bin/wkhtmltopdf
RUN chmod a+x /usr/bin/wkhtmltopdf.sh 
RUN ln -s /usr/bin/wkhtmltopdf.sh /usr/local/bin/wkhtmltopdf

# Now Run gems 
RUN gem install compass
RUN gem install susy

# Git Clone the CV FrameWork from github.
RUN mkdir -p /opt/Code/
RUN git clone https://github.com/barraq/pandoc-moderncv.git  /opt/Code/pandoc-moderncv

# Now begin to build the cv, using the demo 'scaffold'
RUN cd /opt/Code/pandoc-moderncv/ && make scaffold && make pdf HTMLTOPDF=wkhtmltopdf

# Run http server on server 5177, since in dist/ folder we will have the html and pdf
EXPOSE 5177
CMD ["twistd", "-n", "web", "-p", "5177", "--path", "/opt/Code/pandoc-moderncv/dist/"]

```

Put it on github, and trigger an auto-build on dockerhub, pulling it and you could get
the well built docker image.    

### Use this Container
Use it via:    

```
$ sudo docker build -t mycv/mycvapp /home/dash/Code/DockerBuild
$ sudo docker run -d -p 5000:5177 mycv/mycvapp
```

Since our Docker Container listens 5177 port, we use `-p` for mapping local machine's
5000 port to 5177, visit localmachine:5000 then you could found the CV based directory.    


### Change to Debian Based
Maybe Debian based will be much more slim? But this involves the pandoc issue.     


```
# Based on Debian Wheezy
FROM debian:wheezy

# Install Packages, via apt-get. 
RUN apt-get update && apt-get install -y \
        build-essential \
        cabal-install \
        wkhtmltopdf \
        xvfb \
        ttf-wqy-zenhei \
        git \
        rubygems-integration \
        ruby-dev \
        libimage-exiftool-perl \
	zlib1g-dev \
	libdigest-perl \
        python-twisted && rm -rf /var/lib/apt/lists/*

# Via cabal for installing pandoc, latest one will have the markdown plugins
RUN cabal update && cabal install pandoc

# After install pandoc via cabal, update the $PATH
ENV PATH /root/.cabal/bin:$PATH

# Now Change wkhtmltopdf
RUN echo 'xvfb-run --server-args="-screen 0, 1024x768x24" /usr/bin/wkhtmltopdf $*' >
/usr/bin/wkhtmltopdf.sh
RUN chmod a+x /usr/bin/wkhtmltopdf
RUN chmod a+x /usr/bin/wkhtmltopdf.sh 
RUN ln -s /usr/bin/wkhtmltopdf.sh /usr/local/bin/wkhtmltopdf

# Now Run gems 
RUN gem install compass
RUN gem install susy

# Git Clone the CV FrameWork from github.
RUN mkdir -p /opt/Code/
RUN git clone https://github.com/barraq/pandoc-moderncv.git  /opt/Code/pandoc-moderncv

# Now begin to build the cv, using the demo 'scaffold'
RUN cd /opt/Code/pandoc-moderncv/ && make scaffold && make pdf HTMLTOPDF=wkhtmltopdf

# Run http server on server 5177, since in dist/ folder we will have the html and pdf
EXPOSE 5177
CMD ["twistd", "-n", "web", "-p", "5177", "--path", "/opt/Code/pandoc-moderncv/dist/"]

```
