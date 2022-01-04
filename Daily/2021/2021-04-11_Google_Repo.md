# 2021-04-11 (Google Repo)

### Repo란

내가 이해한 Google Repo는 github, gerrit 등의 원격 저장소 git repository 를 한 번에 관리할 수 있는 것이다.



### Repo Manifest 파일

manifest 파일 (.xml 확장자 파일) 에 git repository, branch, commit id 등의 정보를 적어두면, repo 를 생성할 때 git 들의 정보를 manifest 파일을 참조해서 가져오게 된다. 그래서 다른 곳에 git들을 한 번에 배포하거나 할 때 버전 관리를 하기에 용이했다.

```xml
<?xml version="1.0" encoding="UTF-8"?>                                           
<manifest>                                                                       
    <remote                                                                      
        name="origin"                                                            
        fetch="ssh://git@github.com/"/>                                          
    <remote                                                                      
        name="html5"                                                             
        fetch="ssh://git@github.com/"                                            
        review="review.gerrithub.io"/>                                           
    <default revision="master" remote="origin"/>                                 
                                                                                 
    <project                                                                     
        path="bash"                                                              
        name="twityhwan/bash"/>                                                  
    <project                                                                     
        path="c"                                                                 
        name="twityhwan/C_Practice"/>                                            
    <project                                                                     
        path="html5"                                                             
        name="twityhwan/html5"                                                   
        remote="html5"/>                                                         
</manifest> 
[출처] [git-repo] Google repo로 프로젝트 관리하기|작성자 발그레환이
```

[출처] https://blog.naver.com/muri1004/220959739351



### Repo 명령어

```
repo init
repo sync
repo help
```

[출처] https://source.android.google.cn/setup/develop/repo?hl=ko

