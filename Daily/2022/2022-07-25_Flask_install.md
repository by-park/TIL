# 2022-07-25 (Flask install)

Python Flask 프레임워크 설치 시도

Python 버전 문제인가 싶어 Python 설치도 몇 번 시도해봤지만 아직 해결은 못했다.



### Flask 실행을 위한 기본 코드

```python
from flask import Flask
app = Flask(__name__)

@app.route('/')
def hello_world():
    return 'Hello World!'

if __name__ == '__main__':
    app.run()
```

위와 같이 작성 후 `python hello.py` 로 실행

https://flask-docs-kr.readthedocs.io/ko/latest/quickstart.html

https://dydrlaks.medium.com/flask-hello-world-%EC%B6%9C%EB%A0%A5%ED%95%98%EA%B8%B0-17459c8e97ef

https://jjeongil.tistory.com/1567



### Flask 설치 후 에러

그러나 pip install Flask 을 해보니 아래와 같은 에러가 발생한다.

`Command "python setup.py egg_info" failed with error code 1`

해결 방법은 `pip3 install Flask` 였다.

https://stackoverflow.com/questions/66041522/command-python-setup-py-egg-info



### Flask 기본 코드 실행 후 에러

(1) 그 후에 발생한 에러는 `ModuleNotFoundError: No module named 'dataclasses'` 였다.

```python
from dataclasses import dataclass # 에러 발생한 라인
```
https://github.com/fastai/fastai/issues/867

Dataclasses 는 Python 3.7 부터 내장이라고 한다. (현재 시도중인 환경은 Python 3.6)

그래서 dataclasses 를 설치해주었다. `pip3 install dataclasses`



(2) 그랬더니 이번에는 `ModuleNotFoundError: No module named '_contextvars'` 라는 오류가 발생하였다.

`pip3 install contextvars` 하여 해결하였다.

https://yenaworldblog.wordpress.com/2019/05/29/modulenotfounderror-no-module-named-_contextvars/



(3) typing 에서 `AttributeError: module 'typing' has no attribute '_SpecialForm'`라는 에러가 발생하였다.


```python
class _FinalForm(typing._SpecialForm, _root=True): # 에러가 발생한 라인
```
https://github.com/huggingface/transformers/issues/8638

해결책들을 찾아봤는데, Dataclasses 를 지우라고 했다. 하지만 `pip uninstall dataclasses -y` 로 지우면 없다는 (2) 번 에러가 다시 생긴다...
혹시 몰라서 아래에서 Dataclasses 0.4로 해보고, 0.1 까지도 해봤다. 하지만 해결되지 않았다.
https://pypi.org/project/dataclasses/



### Python 버전 변경

Typing 은 내장 라이브러리인듯 해서 Python 버전문제인가 생각해서 3.7 버전을 설치 시도하였다. (기본 환경 Python 3.6)

Python 3.7 설치 방법

```shell
wget https://www.python.org/ftp/python3.7.8/Python-3.7.8.tar.xz
tar xvf Python-3.7.0.tar.xz
cd Python-3.7.0
./configure
sudo make altinstall
```

https://mans-daily.tistory.com/entry/%EB%A6%AC%EB%88%85%EC%8A%A4UbuntuCentOS-Python37-%EC%84%A4%EC%B9%98-%EB%B0%8F-%ED%99%98%EA%B2%BD%EC%84%A4%EC%A0%95-%ED%95%98%EA%B8%B0

그러나 `python3 -V` 입력시 여전히 버전은 3.6 으로 출력되고 있었다. 아래 게시글에서 알려주는 방법대로 PATH 를 추가했지만 여전히 3.6 을 가리키고 있었다. `export PATH=/usr/local/bin/python3.7:$PATH`
https://somjang.tistory.com/entry/PythonUbuntu%EC%97%90%EC%84%9C-Python37-%ED%99%98%EA%B2%BD%EB%B3%80%EC%88%98-%EC%84%A4%EC%A0%95%ED%95%98%EA%B8%B0bashrc%ED%8C%8C%EC%9D%BC%EC%88%98%EC%A0%95

(위의 방법이 맞는 것인지 모르겠다. python3 의 path 는 변경하면 안 된다는 질문 답변이 있었다. https://askubuntu.com/questions/1348089/how-to-change-python3-path-pointer)

다른 방법을 시도해봤는데 ` sudo update-alternatives --install` 옵션을 썼는데, 아래와 같은 에러 메세지가 나왔다. (나중에 시도하다 알았는데, priority 를 안 써줬기 때문이었다.)

https://www.whatwant.com/entry/Python3-%ED%99%98%EA%B2%BD-%EB%A7%8C%EB%93%A4%EA%B8%B0-%EB%B2%84%EC%A0%84-%EB%B3%80%EA%B2%BD%ED%95%98%EA%B8%B0-in-Ubuntu

```shell
$ ~/dev/my_flask_app$ sudo update-alternatives --install /usr/bin/python python /usr/local/bin/python3.7
update-alternatives: --install needs <link> <name> <path> <priority>

Use 'update-alternatives --help' for program usage information.
```

성공한 버전 (이미 3번까지 있었기 때문에 4번으로 적어줬다)

```shell
$ sudo update-alternatives --install /usr/bin/python python /usr/local/bin/python3.7 4
```

그리고 아래 명령어로 선택해주면 된다.

```shell
$ sudo update-alternatives --config python
There are 4 choices for the alternative python (providing /usr/bin/python).

  Selection    Path                      Priority   Status
------------------------------------------------------------

  0            /usr/local/bin/python3.7   4         auto mode
* 1            /usr/bin/python2.7         1         manual mode
  2            /usr/bin/python3.5         0         manual mode
  3            /usr/bin/python3.6         1         manual mode
  4            /usr/local/bin/python3.7   4         manual mode

Press <enter> to keep the current choice[*], or type selection number: 0
update-alternatives: using /usr/local/bin/python3.7 to provide /usr/bin/python (python) in auto mode
```



### typing 관련 문제 추가 확인

typing 을 호출하는 typing-extensions 의 문제일까 싶어서, `pip3 install typing-extensions` 로 최신 버전을 설치해보았다.
https://pypi.org/project/typing-extensions/#history

그러나 아래와 같은 에러가 발생하였다.

```shell
$ python3 hello.py
Traceback (most recent call last):
  File "hello.py", line 3, in <module>
    from flask import Flask
  File "/home/by/.local/lib/python3.6/site-packages/flask/__init__.py", line 6, in <module>
    from . import json as json
  File "/home/by/.local/lib/python3.6/site-packages/flask/json/__init__.py", line 11, in <module>
    from ..globals import current_app
  File "/home/by/.local/lib/python3.6/site-packages/flask/globals.py", line 4, in <module>
    from werkzeug.local import LocalProxy
  File "/home/by/.local/lib/python3.6/site-packages/werkzeug/local.py", line 5, in <module>
    from contextvars import ContextVar
  File "/home/by/.local/lib/python3.6/site-packages/contextvars/__init__.py", line 4, in <module>
    import immutables
  File "/home/by/.local/lib/python3.6/site-packages/immutables/__init__.py", line 18, in <module>
    from ._protocols import MapKeys as MapKeys
  File "/home/by/.local/lib/python3.6/site-packages/immutables/_protocols.py", line 17, in <module>
    from typing_extensions import Protocol
  File "/home/by/.local/lib/python3.6/site-packages/typing_extensions.py", line 160, in <module>
    class _FinalForm(typing._SpecialForm, _root=True):
AttributeError: module 'typing' has no attribute '_SpecialForm'
```

`pip3 install typing` 도 시도해보았지만 문제는 동일했다.



### Python 3.9 설치 시도

Python 3.9 버전을 설치하고자 하였지만, 계속 에러가 발생하여 잘 설치되지 않았다.

```shell
// 방법 1
$ sudo apt install python3.9

// 방법 2
$ wget https://www.python.org/ftp/python/3.9.5/Python-3.9.5.tgz
$ tar -xvf Python-3.9.5.tgz
$ cd Python-3.9.5/
$ ./configure --enable-optimizations
$ make altinstall
// 및 python 경로 설정
```

https://codechacha.com/ko/ubuntu-install-python39/

https://earthconquest.tistory.com/242



### 추가로 시도해볼 내용

Python 3.7 버전을 python 기본 경로로 설정하고 `pip3 install Flask` 를 다시 해보기