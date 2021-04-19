##  hexo blox source

### install
```
npm install hexo-cli -g
hexo init blog
cd blog
npm install hexo-deployer-git --save
npm install
hexo server
```

### _config.yml
```
deploy:
  type: git
  repo: https://github.com/gppola/gppola.github.io.git
  branch: master
```
