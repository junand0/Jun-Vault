#Linux #UID #GID
# Reference
[UID&GID](https://reakwon.tistory.com/228)
[setresuid](https://reakwon.tistory.com/m/234)

# UID&GID
- ruid (real uid)
	- 실제 사용자 id
	- junyeong
- euid (effective uid)
	- 유효 사용자 id
	- root
- suid (saved uid)
	- ruid ↔ euid 간 switching을 위해 저장해둔 id
예를 들어 특정 파일의 읽기 권한을 staff 그룹의 사용자에 필요에 의해 부여하여 읽기 허용했을 때
ruid = 501 (junyeong) /  euid = 0 (root) / suid = 0 (root) 이 된다.

int **setresuid** (uid_t _ruid_, uid_t _euid_, uid_t _suid_);
권한 제한의 필요에 따라 euid를 root에서 junyeong으로 되돌렸다가 다시 root로 전환해야할 때 root id를 저장해둔 suid (0)를 이용하여 전환할 수 있다. suid가 없다면 501에서 0으로 재전환하는 것은 불가능하기 때문에 여기서 suid의 필요성을 알 수 있다.

- 유저들을 권한에 따라 묶어놓은 그룹이 존재 → GID
- UID와 동일한 원리
	- rgid
	- egid
	- sgid
