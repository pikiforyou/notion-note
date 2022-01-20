# pyenv 사용에 관하여 / 설정법

문제점 : [sudo 쓰지마세요 블로그 글](https://medium.com/@chullino/sudo-%EC%A0%88%EB%8C%80-%EC%93%B0%EC%A7%80-%EB%A7%88%EC%84%B8%EC%9A%94-8544aa3fb0e7)

일단 저의 경우 `sudo easy_install pip` 를 하고 에러가 왕창 떴습니다..ㅠㅠ 
이 권한을 수정하기 위해서는 `sudo pip3 ~ --user ~` 을 써야했습니다. sudo를 직접적으로 써도 되겠지만, 추가적으로 vsc에서 파이썬을 확장할때도 또 파이썬이 깔리는 등 관리가 어렵고, 맥북의 기본 설치된 파이썬을 사용하지 말라는 조언을 봤던 기억이 있어서 pyenv 로 구축하기로 했습니다. 

[맥 내장 파이썬을 사용하는 글에 관하여](https://opensource.com/article/19/5/python-3-default-mac)  
pyenv 사용 참고 블로그 : [pyenv 사용참고 블로그 글](https://jiyeonseo.github.io/2016/07/27/install-pyenv/)

파이썬 가상환경 만드는 방법은 크게 4가지 방법이 있습니다.

- **virtualenv**
    - 가장 일반적이고 정통적이고 깔끔한 방법
    - 근데 원하는 버전의 built-in python 이 있어야합니다.
        - python3.6 을 쓰고 싶으면 내 로컬에 python3.6 이 미리 깔려있어야 하는 점
        - 현재 내 로컬엔 python3.8 이 없으므로 python3.8 환경의 가상환경을 못 만드는 점
- **conda**
    - 솔직히 쉽고 원래 쓰던 방법입니다.
    - 근데 conda 자체가 너무 무겁습니다.. 기본 포함된 라이브러리가 너무 Heavy합니다.
- **pipenv**
    - 다른 방법보다 정교해서 기존에 윈도우환경이 아닌 맥에서 설치할 때 이걸 썼습니다.
    - 근데  [이 블로그 글](https://velog.io/@doondoony/pipenv-101) 에서 최근 비추천의 이유가 있다고 해서 제외했습니다.
- **pyenv**
    - pyenv-virtualenv 라는 것을 별도로 깔아주어야 하지만 버전별로 세세하게 구현 가능하다고 합니다
    - 최근 가장 많이 추천되고 있는 가상환경
  
<br>
그래서 pyenv로 가상환경을 설정해보았습니다
<br><br>

1. 설치 및 업그레이드 

```bash
brew update
brew install pyenv
brew upgrade pyenv
```
<br><br>
2. pyenv 환경변수 설정

최근 Mac의 기본 shell이 zsh 이니 zsh로 작성하였습니다. bash쉘일경우에는 `~/.bash_profile`로 설정해주면 됩니다.  또한 변수중 `eval "$(pyenv init -)"` 으로 작성하라는 글들이 많은데 21년 이후로 변수설정시 path를 찾지못하는 오류가 나기때문에, 이 밑에 있는대로 설정해주면 됩니다.

```bash
vi ~/.zshrc
i ##insert 

export PATH="/root/.pyenv/bin:$PATH"
eval "$(pyenv init --path)" 	  	 
eval "$(pyenv virtualenv-init -)" ##이 line은 사실 brew install pyenv-virtualenv 설치후 적용해야한다.

##작성후 vim 에디터 저장후 종료
:wq

##저장후 쉘 재시작
source ~/.zshrc
exec $SHELL
```
<br><br>
3. pyenv를 통한 필요한 파이썬 파일 설치

```bash
pyenv install --list
pyenv install <version_name> 

pyenv versions ##설치된 pyenv 확인
```
원하는 버전으로 각각 설치해주면 된다  
(사실 맥북에 기본으로 깔려있긴한데, 경로를 /usr/bin 으로 사용하지 않기위해 pyenv 환경에서 다시 설치하였다.

<br><br>
💡 3.8.2 설치시 에러가 뜨는경우 (설치가 되지않고 brew openssl 이랑 충돌나는경우)
아래를 터미널에서 다시 실행하면 정상적으로 해결된다.
    
```bash
CFLAGS="-I$(brew --prefix openssl)/include -I$(brew --prefix bzip2)/include -I$(brew --prefix readline)/include -I$(xcrun --show-sdk-path)/usr/include" LDFLAGS="-L$(brew --prefix openssl)/lib -L$(brew --prefix readline)/lib -L$(brew --prefix zlib)/lib -L$(brew --prefix bzip2)/lib" pyenv install --patch 3.8.2 < <(curl -sSL https://github.com/python/cpython/commit/8ea6353.patch\?full_index\=1)
```
<br><br>

4. virtualenv 설치 (가상환경 설치)

```bash
brew install pyenv-virtualenv
pyenv virtualenv 3.8.2 {datepopserver} 

##가상환경 리스트 보기
pyenv virtualenvs
```

원하는 이름으로 만들어주면 됩니다.  
이후 각 프로젝트별로 가상환경을 자동으로 설정되게 해주고 싶다면, 아래를 참고해주자.  
로컬로 실행되게 설정해놓으면 따로 활성화를 해줄필요없이, 해당 git clone 받은 파일에 들어가면 자동으로 가상환경도 실행되게 된다.  
아래와 같이 설정하면 편하다.
<br><br>
```bash
cd 프로젝트파일로 들어가기
pyenv local 가상환경서버명
```
<br><br>
5. 활성화/비활성화  
기본적으로 활성화/비활성화를 통해 제어한다.  
```bash
pyenv activate {가상환경이름}  ##활성화
pyenv deactivate {가상환경이름} ##비활성화
```
<br><br>
💡 activate중 오류뜨는 경우

`~/.zshrc` 파일에 환경변수 제대로 설정되었는지 확인하고 ,
`source ~/.zshrc` 적용해서 다시 실행해볼것
만약 이래도 안된다면, path 잡는중 꼬인거라 zsh 설정을 초기화한후 다시 실행해주면 된다
`$ exec /usr/bin/zsh`  


이래도 안된다면 echo 를 이용해서 한번 더 설정한다  
`echo 'eval "$(pyenv init —path)"' >> ~/.zshrc
 echo 'eval "$(pyenv virtualenv-init -)"' >> ~/.zshrc
 source ~/.zshrc`



