# Mac Postfix Mail 2021

메일 발송 테스트를 위해 로컬 맥에 메일 서버를 간단히 설정해 봅니다.

서버는 Postfix, 클라이언트는 Mail을 사용할 것인데 macOS에 이미 설치되어 있습니다.

## Postfix 설정

건들 것이 거의 없습니다.\
일단 그대로 둡니다.

    /etc/postfix/main.cf:

## Postfix 실행

Postfix 를 바로 실행해 봅니다.\
데몬으로 뜹니다.

    $ sudo postfix start

종료하시려면.

    $ sudo postfix stop

## hosts

`/etc/hosts` 에 메일 테스트용 가상 호스트명을 하나 만듭니다.

    127.0.0.1 mail.test

## Virtual Alias 설정

일반적인 메일 계정은 메일 주소와 유닉스 계정이 연결되어 있습니다.\
메일 주소를 많이 생성해서 테스트하기엔 불편합니다.

Virtual Alias는 이 관계를 무시하는 설정을 할 수 있습니다.\
`@mail.test` 도메인으로 들어오는 모든 메일을 한 계정으로 받아보겠습니다.

### virtual-drypot

적당한 이름으로 virtual alias 파일을 만듭니다.

    $ sudo vim /etc/postfix/virtual-drypot

아래 한 줄 넣어줍니다.

    @mail.test drypot

`@mail.test`로 들어오는 모든 메일을 유닉스 계정 drypot 으로 보낸다는 의미입니다.

### main.cf

`main.cf` 를 엽니다.

    $ sudo vim /etc/postfix/main.cf

맨 아래에 아래 한 줄을 추가합니다.

    virtual_alias_maps = hash:/etc/postfix/virtual-drypot

설정은 이게 전부입니다.

### Reload

아래 명령으로 수정한 설정을 데몬에 적용합니다.

    $ sudo postmap /etc/postfix/virtual-drypot
    $ sudo postfix reload

## 메일 발송 테스트

아래 명령으로 메일을 보내봅니다.

    $ mail -s "mail test" drypot@mail.test

메일을 내용을 다 작성했으면 `^D`로 종료합니다.

메일이 들어왔는지 확인합니다.

    $ mail

들어왔을 겁니다.

## 프로그램에서

프로그램에서는 25번 포트로 접근해서 메일을 보내면 됩니다.\
587번을 쓰면 인증을 하려고 해서 귀찮아집니다.

nodemailer 코드 예입니다.

    const transport = nodemailer.createTransport({
      host: 'localhost',
      secure: false,
      ignoreTLS: true,
      port: 25
    })

    transport.sendMail(...)
