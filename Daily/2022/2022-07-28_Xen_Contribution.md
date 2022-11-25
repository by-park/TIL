# 2022-07-28 (Xen Contribution)

### xen contribution 방법

https://xenproject.org/help/contribution-guidelines/

https://wiki.xenproject.org/wiki/Submitting_Xen_Project_Patches

패치 만들기 (patches/feature-v1 폴더는 직접 만들어야 한다.)

```shell
$ git format-patch {previous commit id} --reroll-count=1 --thread --cover-letter -o ../patches/feature-v1 --subject-prefix=PATCH -1
```

cover letter 는 파일을 열어서 직접 수정하였다.

```shell
$ ./scripts/add_maintainers.pl -d ../patches/feature-v1
```

원래는 이 이후에 보내기 위해서 git send-email 을 해야한다. 

```shell
$ git send-email --to xen-devel@lists.xenproject.org ../patches/feature-v1/*.patch --cc="이름 <메일주소>","이름 <메일주소>" --smtp-debug=1
```

xen 에 contribution 하기 위해서는 위와 같이 git send-email 로 패치를 보내야하지만,

smtp 서버를 사용할 수 없어서 문의한 결과 수동으로 보내도 된다는 답변을 받았다.

https://xen.markmail.org/search/?q=boyoun.park#query:boyoun.park+page:1+mid:uu6jxntivs2ivyeq+state:results

https://xen.markmail.org/search/?q=boyoun.park#query:boyoun.park+page:1+mid:zs45gwseuc2qdqvf+state:results

smtp 서버 사용할 수 없을 때 대체재에 대해 git 메일링 리스트에 문의했으나 답변은 받지 못했다.

https://groups.google.com/g/git-users/c/SHClXYs8pbk/m/iur-vjWOAAAJ



### xen 에 contribution 시도 (2022.07.28)

패치 내용: https://github.com/BY1994/TIL/blob/main/Daily/2022/2022-02-22_Xen_initcall.md

email 내용: https://xen.markmail.org/search/?q=boyoun.park#query:boyoun.park+page:1+mid:jixlx7lxp4wcvcni+state:results



### xen mailing list 확인하는 곳

https://xen.markmail.org/search/?q=

https://lists.xenproject.org/archives/html/xen-devel/

https://xenproject.org/help/mailing-list/