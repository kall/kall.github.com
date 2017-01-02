---
layout: post
title:  "docker + jekyll로 github 페이지에 블로그 작성"
date:   2017-01-01 13:56:47 +0900
---

새해를 맞아, 소소한 목표 하나.

거창하게 해봐야 망할게 뻔하니, 쉽게 할 수 있도록 '깃헙 페이지 업데이트'까지만 해보기로.

먼저, 이미 github page는 설정이 되어 있다. 이에 관해서는 다른 좋을글을 먼저 찾아보시라.

1번, 블로그를 추가해보자.

깃헙 페이지에 블로그 쓸 때는 이미 대세라는 jekyll을 써보기로 결정. 하지만 지킬은 루비를 깔아야 하는데, 깔기는 귀찮고......

도커도 한번 써볼겸해서 도커로 해보기로 결정ㅋ

지킬 도커 이미지가 있는가 찾아봤더니 역시나 있다.
https://hub.docker.com/r/jekyll/jekyll/

설치하고
> docker pull jekyll/jekyll

구동해보니 처음 만난 메세지
> docker run jekyll/jekyll

```
Configuration file: none
            Source: /srv/jekyll
       Destination: /srv/jekyll/_site
 Incremental build: disabled. Enable with --incremental
      Generating...
                    done in 0.013 seconds.
 Auto-regeneration: enabled for '/srv/jekyll'
Configuration file: none
    Server address: http://0.0.0.0:4000/
  Server running... press ctrl-c to stop.
```

일단 디렉토리 연결 옵션 추가로 쉘 실행
> docker run --rm -it -v /home/kall/kall.github.io:/srv/jekyll jekyll/jekyll:latest /bin/bash

그 다음은 그냥 설치한것과 똑같이 사용. /srv/ 디렉토리에서
> jekyll new wallwall

새로 만든 blog(wallwall) 디렉토리에서
> jekyll server --watch

뭔가 뜨는 듯 하지만 접속이 안된다(......)

포트 포워딩추가;;
> docker run --rm -p 4000:4000 -it -v /home/kall/kall.github.io:/srv/jekyll jekyll/jekyll:latest /bin/bash

localhost:4000 으로 접속해보니 뭔가 뜬다. 그 다음은 일반적인 jekyll 사용법대로
- _config.yml 파일 수정
- _blog 디렉토리에 이 글 추가ㅋ
- 그리고 github page에 배포

----
요약
- docker pull jekyll/jekyll
- docker run --rm -p 4000:4000 -it -v /home/kall/kall.github.io:/srv/jekyll jekyll/jekyll:latest /bin/bash
  - jekyll new blog
  - jekyll server --watch
  - 환경수정 & 글쓰기
- 배포
  - jekyll build
  - git add
  - git commit
  - git push

ps. github pages에서 자동으로 jekyll 빌드를 해주지만, 내 경우는 직접 업데이트(..)를 하기 때문에, github page저장소에 .nojekyll 파일을 추가한다. 그래야 빌드가 안깨진다.
