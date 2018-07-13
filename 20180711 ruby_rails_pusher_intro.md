## 20180711 Today I Learned

### Mechanism of Pusher revisited

> Compared to AJAX which requests and gets response on my own client, Pusher can get the response to all the clients that has the channel.

1. Client
   - A button DOM clicks > calls  for create model
2. Server
   - Controller takes action > create model
   - After commit, set the event to some function
   - The method triggers pusher with (channel, event, data)
3. Pusher
   - PUSH!
4. Client
   - has authenticated pusher which subscribes to a channel binded with a method 
   - the trigger matches with subscription, executes!





1. 현재 메인페이지(index)에서 방을 만들었을 때 방 참석인원이 0명인 상태.... 1로 자동증가
2. 방제 수정 삭제 경우,  index 에서 적용될 수 있도록 (pusher 사용)

