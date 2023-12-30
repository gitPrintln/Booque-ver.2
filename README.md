# 📚 Booque ver.2
팀 프로젝트(6명) </br>
교보문고/YES24/알라딘과 같은 책 판매 사이트 & 리뷰 커뮤니티 활성화 & 중고책 거래(채팅)
# 사용기술
Java </br>
SQL </br>
Spring Boot </br>
HTML/CSS/Java Script
# 일정
2023/01/16 ~ 2023/02/16 </br>
# 특징
중고 마켓 홈(검색, 중고 상품 CRUD, 이미지 업로드) </br>
웹소켓을 통한 채팅 기능(채팅 내 검색, 읽음/안읽음) </br>
블로그 이미지 업로드 </br>
새 글, 댓글 알림 </br>
마이페이지
# 내가 맡은 기능(요약)
- 웹 소켓을 이용한 유저들간의 채팅(Stomp websocket)
- 마지막 채팅 가져오기
- 보낸(받은) 채팅에 대한 읽음/안읽음 관련 표시(안읽었으면 1, 읽었으면 사라짐)
# 내가 맡은 기능
- 웹 소켓을 이용한 유저들간의 채팅(Stomp websocket)
chat2.js에서 전체적인 웹소켓 연결을 통해 google창과 시크릿창을 통해서 유저1과 유저2의 서로 실시간 대화를 주고 받음을 테스트하고 본격적으로 프로젝트에서 사용할 채팅방을 구현함.
> SecurityConfig.java 일부
```java
    @Override
    public void configureMessageBroker(MessageBrokerRegistry registry) {
        // 메시지를 구독하는 요청 url prefix => 즉 메시지 받을 때 (subscribe, sub)
        registry.enableSimpleBroker("/user");

        // 메시지를 발행하는 요청 url prefix => 즉 메시지 보낼 때 (publish, pub)
        registry.setApplicationDestinationPrefixes("/app");
    }
```
> chat.js 일부
```java
        var stompClient = null;
        var sender = $('#loginUser').val();
        var chatRoomId = $('#chatRoomId').val();
        var seller  = $('#seller').text();
        // invoke when DOM(Documents Object Model; HTML(<head>, <body>...etc) is ready
        $(document).ready(connect());

        function connect() {
            // map URL using SockJS 
            var socket = new SockJS('/chat');
            var url = '/user/' + chatRoomId + '/queue/messages';
            // webSocket 대신 SockJS을 사용하므로 Stomp.client()가 아닌 Stomp.over()를 사용함
            stompClient = Stomp.over(socket);
            // connect(header, connectCallback(==연결에 성공하면 실행되는 메서드))
            stompClient.connect({}, function() { 
                registration();
                autoScroll();
                // url: 채팅방 참여자들에게 공유되는 경로
                // callback(function()): 클라이언트가 서버(Controller broker)로부터 메시지를 수신했을 때(== STOMP send()가 실행되었을 때) 실행
                stompClient.subscribe(url, function(output) { // 메세지에 관한 구독
                    isOnline();
                    // JSP <body>에 append할 메시지 contents
                    showBroadcastMessage(createTextNode(JSON.parse(output.body)));
                    autoScroll();
                });
                }, 
                    // connect() 에러 발생 시 실행
                        function (err) {
                            alert('error' + err);
            });
 
        };

        // WebSocket broker 경로로 JSON 타입 메시지데이터를 전송함
        function sendChat(json) {
            stompClient.send("/app/chat/"+ chatRoomId, {}, JSON.stringify(json));
        }
        
        const btnSend = document.querySelector('#btnSend');
        const messageInput = document.querySelector('#message');

        // 보내기 버튼 클릭시 실행되는 메서드
        btnSend.addEventListener('click', send);
        
        const date = new Date();
            
        function send() {
            var message = $('#message').val();
            autoScroll();
            sendChat({
                // 'chatRoomId': chatRoomId,
                // 'message': message,
                // 'buyerId': buyerId, 
                // 'sellerId': sellerId,
                // 'usedBookId': usedBookId,
                'sender': sender,
                'message': message,
                'sendTime': getCurrentTime()
                });   
            $('#message').val("");
            // 보낸 후 보내기버튼 비활성화
            btnSend.disabled = true;
            btnSend.style.color = "silver";
            
            const userId = document.querySelector('#userId').value;
            sendChatList(userId)
            
        }

        function sendChatList(userId){
            axios.get('/chat/api/list?userId='+ userId)
                .then(response => {
                    
                        sendBychatList(response)
                    })
                .catch(err => {
                    console.log(err)
                })
            
        }

        // (홍찬) 현재 시간 출력(채팅 주고 받을 때 보낸 시각, 받은 시각 출력)
        function getCurrentTime() {
        return new Date().toLocaleString();
        }

        // HTML 형태의 메시지를 화면에 출력해줌
        // 해당되는 id 태그의 모든 하위 내용들을 message가 추가된 내용으로 갱신해줌
        function showBroadcastMessage(message) {
            $("#content").html($("#content").html() + message);
        }
        function createTextNode(messageObj) {
            if(messageObj.sender == sender){ // 채팅을 보내는 사람
                return '<div style="width: 500px;"><div style="text-align: right; margin-left: 480px; width:270px;" id="newHistory"><div style="text-align: right; width:270px;">' +
            messageObj.message+
            '</div><div style="width: 270px; text-align: right: font-size:13px; color:grey;">' +
            messageObj.sendTime +
            '</div><div id="reads" style="color:dodgerblue;">1</div></div></div>';
            } else { // 채팅을 받는 사람
            return '<div id="newResponseHistory" class="alert alert-info"><div style="width: 40px; margin-right: 15px; display: inline-block; float: left;">' +
            '<img class="rounded-circle" width="40" height="40" src="' +
            chatWithImage + 
            '" style="margin-right:10px;"></div><div style="width: 270px; text-align: left; display: inline-block;"><div>' +
            messageObj.message+
            '</div><div style="font-size:13px; color:grey;">' +
            messageObj.sendTime +
            '</div><br/><br/></div></div>';
            }
        }
```

> ChattingController.java 일부
```java
// 메시지 컨트롤러
//    @MessageMapping("/chat/{userName}")
//    public void sendMessage (@DestinationVariable String userName, Chatting chat) {
//        log.info("sendMessage(to={}, chat={})", userName, chat);
//        boolean isExists = UserChatLog.getInstance().getUsers().contains(userName);
//        if (isExists) {
//            simpMessagingTemplate.convertAndSend("/topic/messages/" + userName, chat);
//            
//        }
//    }
```
초기 MessageMapping 형태에 대한 가닥을 잡음
> ChatController.java 일부
```java
    @GetMapping("/chat")
    public void showChatWindow(@AuthenticationPrincipal UserSecurityDto userDto, Integer chatRoomId, Model model) throws IOException {
        log.info("챗창 오픈 Get Mapping");
        
        Integer loginUserId = userDto.getId();
        User loginUser = userRepository.findById(loginUserId).get();
        String loginNickName = loginUser.getNickName();
        // 뷰에 보여 줄 내 정보
        model.addAttribute("loginUser", loginUser);
        
        List<ChatListDto> list = chatService.loadChatList(loginUserId);
        
        // 뷰에 보여 줄 채팅방 정보들(리스트)
        
        List<Chat> myChats = chatRepository.findByBuyerIdOrSellerIdOrderByModifiedTimeDesc(loginUserId, loginUserId);
    }
//    // (홍찬) 내 대화 목록 불러오기
//    @GetMapping("/chat/list")
//    public String openMyChatList(@AuthenticationPrincipal UserSecurityDto userDto, Model model) throws IOException {
//        Integer loginUserId = userDto.getId();
//
//        log.info("잘 도착햇나{}",loginUserId);
//        // 최근에 업데이트된 날짜 순으로 받아온 내가 대화중인 대화들
//        List<Chat> chat = chatRepository.findByBuyerIdOrSellerIdOrderByModifiedTimeDesc(loginUserId, loginUserId);
//        List<String> cl = new ArrayList<>();
//        for (Chat c : chat) {
//            log.info("방번호{}",c.getChatRoomId());
//            cl.add(chatService.readLastThreeLines(c));
//        }
//        
//        model.addAttribute("myChatList" ,cl);
//        return "chat";
//    }
```
채팅 목록 불러오기 형태에 대한 가닥을 잡음

- 마지막 채팅 가져오기

> ChatService.java 일부
```java
    // (홍찬) 마지막 채팅 가져오기
    public String readLastThreeLines (Chat chat) throws IOException {
        // ChatReadDto recentChat = new ChatReadDto();
        String recentMessage = "";
        
        ChatAssist dto = chatAssistRepository.findByChatRoomId(chat.getChatRoomId());
        //dto.setModifiedTime(chat.getModifiedTime());
        
        // fileName 컬럼을 통해 파일의 경로 찾기, 파일 읽기
        String pathName = fileUploadPath + chat.getFileName();
        
        // 1. RandomAcessFile, 마지막 라인을 담을 String, 읽을 라인 수 "r" 읽기모드
        RandomAccessFile chatResourceFile = new RandomAccessFile(pathName, "r");
        int lineCount = 2; // 마지막 2줄을 읽겠다는 내용!

        // 2. 전체 파일 길이
        long fileLength = chatResourceFile.length();

        // 3. 포인터를 이용하여 뒤에서부터 앞으로 데이터를 읽는다.
        for (long pointer = fileLength - 2; pointer >= 0; pointer--) {

            // 3.1. pointer를 읽을 글자 앞으로 옮긴다.
            chatResourceFile.seek(pointer);

            // 3.2. pointer 위치의 글자를 읽는다.
            char c = (char) chatResourceFile.read();
            
            // 3.3. 줄바꿈이 2번(lineCount) 나타나면 더 이상 글자를 읽지 않는다.
            if (c == '\n') {
                lineCount--;
                if (lineCount == 0) {
                    // 원하는 pointer가 위치해 있을 때 "UTF-8"이라는 변환을 한 후 그 줄의 readline으로 읽어옴.
                    recentMessage = new String(chatResourceFile.readLine().getBytes("ISO-8859-1"),"UTF-8");
                    break;
                }
            }
            
            // 3.4. 마지막 채팅을 db에 저장
            dto.setLastChat(recentMessage);
        }
        chatResourceFile.close();
        // 4. 결과 출력
        //System.out.println(lastLine);
        return recentMessage;
    }
```

> ChatController.java 일부
```java
        // 최신 메세지 내용 불러 오기
        List<String> cl = new ArrayList<>();
        for (Chat c : myChats) {
            log.info("방번호{}",c.getChatRoomId());
            cl.add(chatService.readLastThreeLines(c));
        }
            model.addAttribute("recentMessage" ,cl);
            
            // 최신 메세지 내용 불러 오기  Dto에 넣어서 보냄
        for(int i=0; i<list.size(); i++) {
            String last = chatService.readLastThreeLines(myChats.get(i));
            list.get(i).setLastMessage(last);
            
        }
```

- 보낸(받은) 채팅에 대한 읽음/안읽음 관련 표시(안읽었으면 1, 읽었으면 사라짐)

> ChatAssist.java 일부
```java
  @SequenceGenerator(name = "ChatAssist_SEQ_GEN", sequenceName = "ChatAssist_SEQ", initialValue = 1, allocationSize = 1)
  public class ChatAssist {

    @Id
    @GeneratedValue(strategy = GenerationType.SEQUENCE, generator = "ChatAssist_SEQ_GEN")
    private Integer Id;
    
    @Column(nullable = false)
    private Integer chatRoomId;
    
    @Column
    private String nickName; 
    
    @Column
    private Integer readChk; // 읽었으면 0, 안읽었으면 1
    
    @Column
    private Integer readCount;
    
    @Column
    private String lastChat;
    
    @Column
    private LocalDateTime modifiedTime;
    
    public ChatAssist updateLastChat(String lastChat, LocalDateTime modifiedTime) {
        this.lastChat = lastChat;
        this.modifiedTime = modifiedTime;
        return this;
    }
    
    public ChatAssist updateReadChk(String nickName, Integer readChk, Integer readCount) {
        this.nickName = nickName;
        this.readChk = readChk;
        this.readCount = readCount;
        return this;
    }
    
  }
```

> chat.js 일부
```java
      function connect() {
          var url2 = '/user/' + chatRoomId + '/queue/notification/'+ seller;
          stompClient.connect({}, function() { 
                stompClient.subscribe(url2, function(output){ // 메세지 읽음 알림에 관한 구독
                    read(); 
                });
                }, 
                    // connect() 에러 발생 시 실행
                        function (err) {
                            alert('error' + err);
            });
 
        };

        // 채팅 입력창에 포커싱 맞췄을 때 발동 > 상대방이 메세지를 확인(즉, 읽음으로 간주)
        $('#message').focus(function(){
            const nm = document.querySelectorAll('#newResponseHistory');
            for (var i = 0; i < nm.length; ++i) {
            nm[i].className = "";
            nm[i].removeAttribute('id');
            }
            stompClient.send("/app/chat/read/"+ chatRoomId, {}, JSON.stringify(
            {sender:sender}
        ))
        });

        // 대화를 시작하는 대상 등록
        function registration(){
        $.get("/registration/" + sender)
        .fail(err =>(console.log(err)))
        };

        // 상대방이 온라인/오프라인 확인
        function isOnline(){
            const data = {
            nickName: seller,
            chatRoomId: chatRoomId
        }
            $.post("/onlineChk", data)
            .done(response => { 
                if(response == 1){ // true: 상대가 online
                     
                }
            })
            .fail(err => console.log("온라인/오프라인 오류"))
        }

        //read();
        // 읽음 = 0 안읽음 = 1 체크
        function read(){
            const read = document.querySelectorAll('#reads');
            for (var i = 0; i < read.length; ++i) {
            read[i].style.visibility = 'hidden';
            read[i].removeAttribute('id');
            }
            
        }

        function showUnread() {
        let chatRoom = document.querySelectorAll('.btnChatRoom');
        let unread = document.querySelectorAll('.unread');
        let unreadNickName = document.querySelectorAll('.unreadNickName');

        for (i = 0; i <= chatRoom.length; i++) {
                unreadCount = unread[i].value;
                unreadName = unreadNickName[i].value;
                if ((unreadName == sender) && (unreadCount > 0)) {
                    chatRoom[i].style.display = "block";
                } else {
                    chatRoom[i].style.display = "none"; 
                }
            } 
        }
```
> ChattingController.java 일부
```java
    // (홍찬) 메세지 알림을 위해 읽음 푸시
    @MessageMapping("/chat/read/{chatRoomId}")
    public void notification(@DestinationVariable Integer chatRoomId, ChatReadDto dto) throws IOException {
        String nickName = dto.getSender();
        String url2 = "/user/" + chatRoomId + "/queue/notification/" + nickName;
        simpMessagingTemplate.convertAndSend(url2, nickName);
    }

    // 채팅 로그 사용자 등록
    @CrossOrigin
    @GetMapping("/registration/{nickName}")
    public ResponseEntity<Void> register(@PathVariable String nickName){
        log.info("register(username={})", nickName);
        try {
            UserChatLog.getInstance().setUser(nickName);
        } catch (Exception e) {
            e.printStackTrace();
            return ResponseEntity.badRequest().build();
        }
        return ResponseEntity.ok().build();
    }
    
    // 채팅 로그 사용자 등록 취소
    @CrossOrigin
    @GetMapping("/unregistration/{nickName}")
    public ResponseEntity<Void> unregister(@PathVariable String nickName){
        log.info("unregister(username={})", nickName);
        UserChatLog.getInstance().unsetUser(nickName);
        return ResponseEntity.ok().build();
    }
    
    // 채팅하고 있는 상대가 웹소켓에 연결중인지 확인
    @PostMapping("/onlineChk")
    public Integer isExists(String nickName, Integer chatRoomId) {
        log.info("isExists{},{}",nickName,chatRoomId);
        boolean isexist = UserChatLog.getInstance().getUsers().contains(nickName);
        if (isexist) {
            log.info("상대방{} 상태: online입니다.", nickName);
            return 3; // TODO
        } else {
            log.info("상대방{} 상태: offline입니다.", nickName);
            return chatService.updateReadChat(nickName, chatRoomId, 0);
        }
    }
```
> ChatController.java 일부
```java
    @GetMapping("/chat")
    public void showChatWindow(@AuthenticationPrincipal UserSecurityDto userDto, Integer chatRoomId, Model model) throws IOException { 

        // 최신 메세지 내용 불러 오기
        List<String> cl = new ArrayList<>();
        for (Chat c : myChats) {
            log.info("방번호{}",c.getChatRoomId());
            cl.add(chatService.readLastThreeLines(c));
        }
            model.addAttribute("recentMessage" ,cl);
            
        // 최신 메세지 내용 불러 오기  Dto에 넣어서 보냄
        for(int i=0; i<list.size(); i++) {
            String last = chatService.readLastThreeLines(myChats.get(i));
            list.get(i).setLastMessage(last);
            
        }
             // 안읽은 메세지 개수
             ChatAssist CA = chatAssistRepository.findByChatRoomId(myChats.get(0).getChatRoomId());
             Integer unreadCount = CA.getReadCount();
             String withName = CA.getNickName();
             if(unreadCount != 0) {
                 if (loginNickName.equals(withName)) {
                     unreadCount = chatService.updateReadChat(withName, myChats.get(0).getChatRoomId(), 1);
                     model.addAttribute("rcount", unreadCount);
                 } else {
                 model.addAttribute("rcount", unreadCount);
                 }
                 } else {
                     model.addAttribute("rcount", 0);
                 }

            // 안읽음 리스트를 위해 전달.
            List<UnreadDto> unreadInfo = new ArrayList<>();
            Integer unread = 0;
            String unreadNickName = "";
            for (Chat c : myChats) {
                unread = chatAssistRepository.findByChatRoomId(c.getChatRoomId()).getReadCount();
                unreadNickName = chatAssistRepository.findByChatRoomId(c.getChatRoomId()).getNickName();
                UnreadDto dto = UnreadDto.builder().chatRoomId(c.getChatRoomId()).unread(unread).unreadNickName(unreadNickName).build();
                unreadInfo.add(dto);
            }
            model.addAttribute("unreadInfo", unreadInfo);
    }
```
> ChatService.java 일부
```java
    // 안읽은 메세지 저장(안읽음=1, 읽음=0)
    public Integer updateReadChat(String nickName, Integer chatRoomId, Integer unread) {
        ChatAssist entity = chatAssistRepository.findByChatRoomId(chatRoomId);
        log.info("ncu={}{}{}",nickName, chatRoomId,unread);
        // 메세지를 읽은 경우| nickName : 기준이 되는 사람 닉네임, unread : 안읽었으면 1/읽었으면 0
        if (unread == 1 && nickName.equals(entity.getNickName())) { // 읽었을 경우
            entity.updateReadChk(nickName, 0, 0);
            return 0;
        }
        // 메세지를 안읽은 경우
        if (entity.getReadCount()!= 0) { // 안읽은 메세지 개수 갱신하는 경우
            if (nickName.equals(entity.getNickName())) { // 닉네임이 같으면 안읽음개수 추가
                entity.updateReadChk(nickName, unread, entity.getReadCount()+1);
                return 1;
            } else { // 닉네임이 다르면 유저 갱신후 안읽음 1 추가(닉네임이 다를 때: 처음 만들어지는 경우)
                entity.updateReadChk(nickName, unread, 1);
                return 1; // 안읽음이 역전되었으니 상대방 입장에서 읽음.
            }
        } else { // 처음 만들어지는 경우
            entity.updateReadChk(nickName, unread, 1);
            return 1;
        }
    }
```

