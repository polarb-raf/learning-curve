# 머신 한대에서 두 개의 github 계정 사용하기

회사에서 지급 받은 노트북을 개인 노트북 겸용으로 쓰면서 회사와 개인 github계정을 한 컴퓨터에서 사용하게 되었다.
그래서 레포에 맞게 계정을 따로 사용해야 하는 상황이 생겼고, 번거로움을 조금 줄이기 위해 노력한 과정을 정리해본다.

> macOS Catalina(10.15.3) 기준  

### 내용 요약
1. 작업하는 Repository에 따라 각 계정에 맞는 올바른 ssh key 사용하도록 설정하기 (feat. ssh key 만들기)
2. 작업하는 폴더의 '경로'에 따라 알맞은 name과 email 주소를 사용하기

### ssh key 생성

> ssh key를 생성하는 방법은 github에서도 안내해주고 있다. [Link](https://help.github.com/en/github/authenticating-to-github/generating-a-new-ssh-key-and-adding-it-to-the-ssh-agent)  

1. 터미널에 다음 명령어를 입력하여 ssh key 생성을 시작한다.
   
```shell script
# 키를 등록할 본인 깃헙 계정의 이메일을 적는다. 꼭 그래야 하는지는 확인해보지 못했음
$ ssh-keygen -t rsa -b 4096 -C "your_email@example.com"
```

2. 터미널에 다음 문장이 뜨면 키를 저장할 경로와 저장할 키 이름을 입력한다.
    - 각 계정당 키를따로 생성해야 하므로 나는 키 이름을 `id_rsa_이름`, `id_rsa_회사이름` 이렇게 만들었다
    - 이후로는 키 이름을 편의상 `id_rsa_foo`, `id_rsa_bar`로 표기한다.
```shell script
> Enter a file in which to save the key (/Users/you/.ssh/id_rsa): [Press enter]
# /Users/컴퓨터 계정명/.ssh/id_rsa_myname 이런 식으로 저장했다
```

3. ssh 키의 비밀번호를 입력한다.
    - 필수는 아니다. 아무 것도 입력하지 않고 엔터를 치면 비밀번호가 없는 상태로 저장된다
```shell script
> Enter passphrase (empty for no passphrase): [Type a passphrase]
> Enter same passphrase again: [Type passphrase again]
```

4. 생성된 키를 ssh-agent를 실행시키고 등록한다.

```shell script
#agent 실행
$ eval "$(ssh-agent -s)"
> Agent pid *****

#key 등록

$ ssh-add -K ~/.ssh/id_rsa_foo
```

5. 계정을 두개를 사용하기 위해 키를 만드는 과정을 한번 더 반복한다.
    - 마치면 ~/.ssh 폴더 안에 아래와 같이 파일이 생성되었을 것이다. .pub가 붙어있는 파일이 공개키 파일이다.

> id_rsa_foo  
> id_rsa_foo.pub  
> id_rsa_bar  
> id_rsa_foo.pub

6. 깃헙 계정에 공개키를 등록한다.
    - 깃헙 계정 로그인 후 우상단 아이콘을 눌러서 Settings로 들어가면 `SSH and GPG keys`라는 메뉴로 들어간.
    - New SSH Key 버튼을 눌러 새 키를 생성하는 과정을 시작한다.
    - Title은 본인이 알아보기 쉬운 걸로 아무거나 입력해도 좋다.
    - Key입력하는 곳에 본인이 등록할 공개키의 내용을 입력한다.
```shell script
# 에디터로 열어서 공개키의 내용을 복사할 수 있지만 아래와 같이 입력하여 복사할수도 있다.
pbcopy < ~/.ssh/id_rsa_foo.pub
```

7. ~/.ssh 폴더에 ssh 설정 파일을 만든다.
    - 설정파일은 보통 .ssh 폴더 내에 config라는 이름으로 저장되어 있다. 이걸 보며 ssh 키를 처음 만들었다면 해당 파일이 없을 수도 있는데, 새로 생성해주면 된다.
```shell script
Host foo-github.com
  HostName github.com
  IdentityFile ~/.ssh/id_rsa_foo
  User blahblahblah #이 키를 등록한 github 계정의 username(이메일이 아님)을 적었다. 아래도 동일 

Host bar-github.com
  HostName github.com
  IdentityFile ~/.ssh/id_rsa_bar
  User blahblah
```
> foo-github.com, bar-github.com 이 부분은 본인이 넣고싶은 아무 텍스트나 넣어도 상관이 없다.
    
8. ssh 키로 깃허브를 이용할때는 git 주소를 https 프로토콜이 아닌 다른 방식으로 이용해야 한다.
    - 이미 기존에 사용하던 레포도 ssh 키를 이용하려면 remote의 주소를 바꿔줘야 한다.
    - ssh 방식으로 사용할 때는 이런 주소를 사용한다 -> git@github.com:username/your_repository.git
    - 위 주소에 github.com 부분을 본인이 ssh 설정파일에 넣어둔 Host로 바꾸어준다.
```shell script
example: git@foo-github.com:foo/foo_repo.git

#새로 클론을 받을 때
git clone git@foo-github.com:foo/foo_repo.git

#기존에 받아서 사용하던 레포의 remote 주소를 바꿀 때
git remote set-url origin git@foo-github.com:foo/foo_repo.git
```

9. 여기까지 완료했다면 레포마다 계정을 바꿔서 로그인해가며 작업하지 않아도 된다. 

### 작업 '경로'에 따라 알맞은 user.name, user.email 사용하게 만들기

> ssh 키만 등록한다면 권한에 맞는 키가 알아서 사용되겠지만 gitconfig가 분리되지 않아서 커밋하거나 할 때 올바른 메일 주소와 이름이 들어가지 않을 수 있다. 그걸 해결하기 위한 작업.  
> 저장소별 로컬 config를 매번 지정해주는 것도 가능하지만, 작업 흐름상 새로 클론아서 작업하거나 하면 잊고 작업하는 경우가 많다.

1. ~/.gitconfig 파일을 열어서 수정한다.
    - 기본적인 컨셉은 작업 '경로'에 따라 gitconfig를 추가로 불러와서 기존의 값을 덮어씌우는 방식이다.
    - 나는 회사 업무에 쓰는 저장소는 모두 한 폴더 안에 넣어두기 때문에 쉽게 덮어쓸 수 있었다.
    
 ```shell script
# ~/.gitconfig 파일의 내용
# 여기에 명시된 user 정보를 기본으로 사용하지만
# ~/YourWorkingDirectory/ 경로에서 작업한다면 .gitconfig-working이라는 파일을 추가로 include 하겠다는 내용이다.

[user]
	name = myName
	email = my@email.com
[includeIf "gitdir:~/YourWorkingDirectory/"]
  path = .gitconfig-working
```
     
2. ~/.gitconfig-working 파일을 생성한다.
    - .gitconfig에 include할 파일의 이름을 .gitconfig-working으로 했기 때문에 파일 이름을 맞출 뿐, 꼭 저 파일명은 아니어도 된다.
```shell script
# ~/.gitconfig-working 파일
# gitconfig의 세팅이 모두 유지되면서 겹치는 내용만 덮어씌워지게 된다.
[user]
  name = myName
  email = workingMail@email.com
```


### 참고
[깃헙 공식 help 문서](https://help.github.com/en/github/authenticating-to-github/generating-a-new-ssh-key-and-adding-it-to-the-ssh-agent#adding-your-ssh-key-to-the-ssh-agent)
[outsider 님의 블로그](https://blog.outsider.ne.kr/1448)