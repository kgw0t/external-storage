# external-storage
## Node

```
$ brew install nodenv

# 下記を ~/.zshrc に記載
# eval "$(nodenv init -)"
$ vi ~/.zshrc
$ source ~/.zshrc

$ nodenv install -l
$ nodenv install 15.11.0
$ nodenv global 15.11.0
$ nodenv versions
...
* 15.11.0 (...)
```

## Hexo
https://hexo.io/

```
$ npm install hexo-cli -g

$ hexo init blog
$ cd blog
$ npm install

$ hexo clean
$ hexo generate

$ hexo server
```
