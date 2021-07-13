# 2021-07-12 (Repo branch)

google repo 의 branch 형태가 다음과 같았는데, 일반적으로 보던 origin/HEAD -> origin/master 형태가 아니라서 이게 무슨 뜻인지 찾아보려고 했다. 결과적으로 정리된 글은 못 찾았는데, 다음과 같이 보인다.

```shell
$ git branch -r
m/repo_branch -> by/dev
by/dev
by/topic
by/master
```

위와 같이 repo로 보지 않고, 개별 git clone을 하면 다음과 같이 보이게 된다.

```shell
$ git branch -r
origin/HEAD -> origin/master
origin/dev
origin/topic
origin/master
```

(HEAD와 master에 대해 설명한 문서는 있다. https://git-scm.com/book/ko/v2/Git-%EB%B8%8C%EB%9E%9C%EC%B9%98-%EB%B8%8C%EB%9E%9C%EC%B9%98%EB%9E%80-%EB%AC%B4%EC%97%87%EC%9D%B8%EA%B0%80)



repo 라는 git 에 repo_branch 라는 branch가 있다.

```shell
repo init -u ssh://주소/by/repo -b repo_branch --repo-url=ssh://주소/tools/repo
```

repo 안 default.xml 파일에 각 개별 git 마다 어떤 branch 사용할지 정해준다.



repo/default.xml 이고, 이건 .repo/manifest.xml 파일로 이어진다.

default.xml 이 있는 repo 는 repo_branch 로 checkout 한다. 그러면 아래와 같은 파일을 만들어둔 것이 보일 것이다.

```xml
<?xml version="1.0" encoding="UTF-8"?>

<manifest>
    <default revision="dev" sync-j="16" />
    <remote name = "by" fetch="ssh://주소/my_git" revision="dev" />
    
    <!-- yocto meta layers -->
    <project remote="by" name="by_dev" path="sources/by_dev"/>
</manifest>
```

여기서 정리된 project와 branch 정보가 repo 안에 개별 git 들을 구성할 때 사용하게 되는 것이다.

그러면 처음에 봤던 m/repo_branch -> by/dev는 repo 의 repo_branch 의 default.xml 이 가리키는 branch 정보는 by/dev 입니다 (remote 명 / branch 명) 로 보면 될 것 같다.