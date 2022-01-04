# 2021-09-24 (Github SSH)

새로 repository 를 만들어서 연결하려고 했더니 Github 에서 SSH key 를 요구했다. 자꾸 문제가 생겨서 아래 링크를 따라했는데, 자세해서 이번엔 성공하였다. 해당 환경에서 `ssh-keygen` 을 하고  `.ssh/config` 파일까지 등록해주니까 연결되었다.

ssh 키 생성 (.ssh 경로 밑에서 파일 이름 원하는 거로 설정한 후에 비밀번호는 엔터를 쳤다)

```shell
$ ssh-keygen -t rsa -b 4096 -C "your_email@example.com"
Generating public/private ed25519 key pair.
Enter file in which to save the key (/root/.ssh/id_ed25519): {여기에 원하는 파일 이름을 적으면 된다.}
Enter passphrase (empty for no passphrase):
Enter same passphrase again:
```

.ssh/config에 아래를 등록하였다.

```shell
Host github.com
  IdentityFile ~/.ssh/id_ed25519 {여기는 실제 위에서 만든 파일 경로로 변경}
  User git
```

그리고 git 도 git remote add 할 때 https 말고 ssh 로 가져왔다.