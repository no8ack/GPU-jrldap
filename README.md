## Openldap

1. **서버 설치**
```
$ sudo apt-get install -y slapd ldap-utils
```
```
$ sudo dpkg-reconfigure slapd
```

  reconfigure을 통해서 amdin의 cn=과 admin passwd를 설정을 다시 해줘야 합니다.
  
```
$ sudo apt -y install apache2 php php-cgi libapache2-mod-php php-mbstring php-common php-pear
```
```
$ sudo apt-get install phpldapadmin
```

php를 설치하는 이유는 웹 페이지에서 user를 관리하기 위해서 입니다.
  
2. **phpldapadmin 설정** (300번째 줄 정도에서 수정하면 됩니다.)

servers->setValue('server','host','127.0.0.1');

servers->setValue('server','base',array('dc=naver,dc=com'));

servers->setValue('login','bind_id','cn=admin,dc=naver,dc=com'); 

```
$ /etc/init.d/apache2 restart
```

 <a href="http://127.0.0.1/phpldapadmin">http://127.0.0.1/phpldapadmin</a> 

 위의 사이트를 통해서 user관리를 할 수 있습니다.

 (참고 : <a href="https://dejavuqa.tistory.com/343?category=303713">https://dejavuqa.tistory.com/343?category=303713</a>)

3. **host ip 설정**
```
$ vi /etc/hosts
``` 
에 본인의 ip(127.0.0.1)와 이름(localhost00)을 추가해주면 됨.
</ol>

4. **클라이언트 설치**
```
$ sudo apt-get -y install libnss-ldap libpam-ldap ldap-utils nscd
```

 설치 시에 server의 주소를 입력해야 하는데 위의 server에서 사용한 주소(127.0.0.1)를 입력하면 됩니다.
그리고 ldapi:// -> ldap:/로 고쳐야 합니다.

5. **비밀번호 설정**
 ```
$ sudo vi /etc/nsswitch.conf
```

여기에서 아래 3줄을 수정해주면 됩니다.
passwd : compat ldap
group  : compat ldap
shadow : compat ldap

 ```
$ sudo vi /etc/pam.d/common-session
```  
에서
```
$ session optional	pam_mkhomedir.so
``` 
를 추가해줘야 합니다.

user 로그인시에 홈 디렉토리를 만들어줍니다.

설정이 끝났다면 재실행을 해줍니다.

 ```
$ sudo /etc/init.d/nscd restart 
```
```
$ sudo /etc/init.d/nslcd restart
```

 phpldapadmin에 접속해서 user계정을 새로 만들었다면 

 ```
$ getent passwd
```
 를 통해서 확인 가능합니다.


_ _ _

## Nfs 설정

```
$ apt-get update
```
```
$ apt-get install nfs-common nfs-kernel-server portmap
```
1. **공유 디렉토리 생성**
```
$ mkdir /home/nfs
```
```
$ chmod 777 /home/nfs
```
```
$ vi /etc/exports 
```
```
/home    123.456.78.910(rw, sync)
```
위의 설정은 NFS 서버의 특정 IP의 호스트 접속을 허용하는 설정입니다.
```
rw : 읽기, 쓰기 가능
ro : 읽기만 가능
secure : 클라이언트 마운트 요청시 포트를 1024 이하로 합니다.
noaccess : 액세스 거부
root_squach : 클라이언트의 root가 서버의 root권한을 획득하는 것을 막습니다.
no_root_squash : 클라이언트의 root와 서버의 root를 동일하게 합니다.
sync : 파일 시스템이 변경되면 즉시 동기화합니다.
all_squach : root를 제외하고 서버와 클라이언트의 사용자를 동일한 권한으로 설정합니다.
no_all_squach : root를 제외하고 서버와 클라이언트의 사용자들을 하나의 권한을 가지도록 설정합니다. 
```
2. **NFS 실행**
```
$ /etc/init.d/rpcbind start
```
```
$ /etc/init.d/portmap start
```
```
$ /etc/init.d/rpcidmapd start
```
```
$ /etc/init.d/nfs start
```
3. **NFS 클라이언트 설정**
마운트해서 사용할 디렉토리 생성
```
$ /mnt/nfs
```
NFS 서버와 마운트
```
$ mount -t nfs [호스트명 또는 IP]:[/NFS 서버에서 설정한 공유 디텍터리] [/마운트 포인트]
```
부팅시 마운트를 시키고자 할 때
```
/etc/fstab
```
에 등록.
