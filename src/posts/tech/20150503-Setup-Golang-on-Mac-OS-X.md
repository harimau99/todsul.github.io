## Setup Go workspace

    mkdir $HOME/go
    cd $HOME/go
    mkdir bin pkg src

## Setup Go environment

    sudo vi ~/.bash_profile
    export GOPATH=$HOME/go
    PATH=$PATH:$GOPATH/bin

## Create, build and run a Go project

    mkdir $GOPATH/src/github.com/todsul/hello
    touch $GOPATH/src/github.com/todsul/hello/hello.go
    go install
    hello