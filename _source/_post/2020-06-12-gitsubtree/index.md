---
title: Github Subtree 로 프로젝트 임베딩
author: langley
---

# Abstract

블로그 프로젝트를 진행하면서 [Builder](https://github.com/langley0/bellog) 와 [Markdown](https://github.com/langley0/langley0.github.io/) 으로 프로젝트를 분리하였다. 목적은 Builder 를 모듈화해서 여러가지 블로그 타입을 생성하기 위해서였다 ~~아무래도 이런일은 없을 것 같다~~.  
이런 상황에서 블로그를 빌드하기 위해서는 Builder 프로젝트를 손쉽게 실행 할 수 있는 방법이 필요했다

가장 일반적인 방법은 npm library 로 등록하는 것이었지만, 난 이 프로젝트가 완성된 것이 아니기에 공개하고 싶지는 않았다. 다음 방법은 사용자가 직접 Builder 프로젝트를 클론과 같은 방법으로 받아서 블로그 빌드패스를 연결하는 것이다. 가장 쉬운 방법이지만 뭔가 해냈다라는 기분이 들지 않아서 일단은 패스.

마지막으로는 프로젝트를 다른 프로젝트에 임베드 시키는 [git subtree](https://github.com/git/git/blob/master/contrib/subtree/git-subtree.txt) 를 활용하는 것이다

# Git-Subtree
### Description
서브트리는 프로젝트를 다른프로젝트의 서브폴더 형태로 포함시키는 역활을 한다.

서브트리는 비슷한 일을 하는 서브모듈과 혼동되는 경우가 많다. 서브모듈과는 다르게, 서브트리는 *.gitmodules 파일이나 gitlinks 명령같은* 별도의 초기화 작업은 필요로 하지 않는다.  그리고 메인 프로젝트의 참여자들이 서브트리의 동작에 대해 이해할 필요가 없다. 서브트리에도 동일하게 커밋, 브랜치, 머지등의 작업을 수행할 수 있다.

다른 프로젝트를 머지하는 것과 서브트리를 만드는것사이에 큰 차이점이 있다. 우선 서브트리의 원래 프로젝트가 업데이트되었다면, 손쉽게 현재 프로젝트에 변경점을 받아올 수 있다. **거기다 별도의 작업없이 바로 적용된다**. 반대로 현재 서브트리 폴더안의 변경점을 원래 라이브러리 프로젝트에 적용할 수도 있다. 마지막으로 현재 폴더를 다른 독립된 라이브러리로 만드는 것도 가능하다.

예를 들어, 어플리케이션을 만들었는데, 여기에 사용한 라이브러리가 매우 유용하다고 생각되면, 이 라이브러리 (서브폴더) 에 대한 히스토리를 그대로 분리해서 독립된 프로젝트로 만들 수 있게 되고, 이러한 방법을통해서 어플리케이션 작업의 히스토리가 섞여 들어오는 것을 막을 수 있다.

> [TIP]
> 서브트리와 메인 프로젝트의 커밋을 최대한 분리하는 것을 추천한다. 라이브러리와 메인에 동시 적용되는 내용이 있다고 하더라도 이것을 두개의 커밋으로 쪼개어서 작업하도록 하자. 이런 작업을 통해서 이후에 라이브러리를 분리하게 될때, 커밋들이 이해가능한 상태를 유지하게 된다. 단 이작업은 필수는 아닌데, 깃은 서브폴더에 변경점이 없는 커밋에 대해서는 이후에 서브트리로 분리할때 자동으로 제외시켜준다.



### Synopsis
``` bash
git subtree add   -P <prefix> <commit>
git subtree add   -P <prefix> <repository> <ref>
git subtree pull  -P <prefix> <repository> <ref>
git subtree push  -P <prefix> <repository> <ref>
git subtree merge -P <prefix> <commit>
git subtree split -P <prefix> [OPTIONS] [<commit>]
```

### Commands
**add**: 입력된 <commit> 이나 <repository> 에서 프로젝트를 가져와서  <prefix> 이름의 서브트리를 생성한다.
**merge**: <commit> 까지의 모든 변경점을 <prefix> 서브트리에 적용한다. 이 작업은 로컬 변경점을 제거하지 않는다.
**pull**: `git pull` 과 비슷하게 원격 저장소에서 변경점을 가져온다.
**push**: 로컬변경점을 주어진 리파지터리에 추가시킨다. 브랜치변경도 가능하다.
**split**: <prefix> 의 서브트리를 새로운 프로젝트로 분리시킨다. 

### Example
**Add command**
외부 라이브러리를 현재 프로젝트에 git-subtree 라는 폴더에 서브트리로 추가해보자
``` bash
$ git subtree add --prefix=git-subtree --squash \
    git://github.com/apenwarr/git-subtree.git master
```
--squash 플래그는, 기존의 로그를 하나로 압축해서 기록한다. 프로젝트의 모든 히스토리를 보고 싶지 않은 경우에 유용하다. 이 명령을 사용하더라도 커밋자체가 손실되지는 않기 때문에 이후 변경점을 적용하는데에 문제없다.

**Extract a subtree with using commit**
먼저 git 소스코드를 사용해서 작업을 진행해보자
``` bash
$ git clone git://git.kernel.org/pub/scm/git/git.git test-git
$ cd test-git
```

이안에 더이상 유지보지하지않는 gitweb 이 있는데, 이것을 별도의 프로젝트로 분리해보자
``` bash
$ git subtree split --prefix=gitweb --annotate='(split) ' \
     0a8f4f0^.. --onto=1130ef3 --rejoin --branch gitweb-latest
$ gitk gitweb-latest
$ git push git@github.com:whatever/gitweb.git gitweb-latest:master
```

위에서 "0a8f4f0^.." 라는 표현을 사용하였는데, 이것의 의미는 "0a8f4f0부터(자신을 포함) 최신까지의 모든 변경점" 이라는 뜻이다

만약에 gitweb  `git subtree add` 로 추가되어있다면 별도의 커밋 번호 없이 split 을 할 수 있다.
``` bash
$ git subtree split --prefix=gitweb --annotate='(split) ' \
    --rejoin --branch gitweb-latest2
```

이제 변경점을 만들어보자.
``` bash
$ date >gitweb/myfile
$ git add gitweb/myfile
$ git commit -m 'created myfile'
```
`date` 명령에 의해서 다음과 같은 내용이 적힌 파일을 생성하게 된다
> 2020년 6월 12일 금요일 오후 1:41:23

이제 다시 브랜치를 업데이트 해보면 파일이 온전히 있는 것을 볼 수 있다
```bash
$ git subtree merge --prefix=gitweb --squash gitweb-latest
```

**Extract a asubtree with branch**
디렉토리안에 파일들이 복잡하게 얽혀있거나 하는 상황에서 서브트리를 통해 프로젝트를 분리할때에 필요한 방법이다
먼저 리파지터리를 준비한다
``` bash
$ mkdir <new-repo>
$ cd <new-repo>
$ git init --bare
```

다시 원래 소스가 있는 폴더로 돌아와서 브랜치를 새로 준비한다
``` bash
$ git subtree split --prefix=lib --annotate="(split)" -b split
```

준비된 브랜치를 새로운 리파지터리에 추가한다
``` bash
$ git push <new-repo> split:master
```