![600](https://i.imgur.com/E9izYPz.png)

client $\to$|browser| DNS server $\to$ Load Balancer : server 선택(scheduling) $\to$ Caching service ; + DB $\to$ 

- Sync web server
	- 각 client request or connect 당 process or thread를 복제하여 할당하고 처리
	- Pros
		- 서버 사양이 충분하여 많은 thread를 감당할 수 있을 때 좋음
		- request 수가 적고 크기가 클 때 좋음
	- Cons
		- 입출력 요청 처리가 끝날 때까지 메모리 버퍼 차지 $\to$ 메모리 낭비
- Async
	- request와 processing에 대해서 thread 따로 관리
	- 