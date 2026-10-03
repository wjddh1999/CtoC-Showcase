# 수신 큐와 Unity 프레임 전달

수신 JSON과 정규화한 채팅 데이터를 각각 lock으로 보호한 FIFO 큐에 넣고 Update에서 처리합니다. JSON 기본 응답은 즉시 이벤트로 전달할 수 있고 유연 파싱 경로는 채팅 큐를 거칩니다. 모든 경로가 반드시 두 큐를 통과하는 구조는 아닙니다.

**본인 역할:** 수신 큐·디스패치 구현. STOMP 전송 계층은 외부 라이브러리를 사용합니다.

**형식:** 일부 발췌. 독립 실행 파일이 아니며 클래스 필드와 나머지 메서드를 포함하지 않습니다.

**의존성:** Unity MonoBehaviour, System.Collections.Generic, ChatMessageData, 큐·잠금 필드, HandleChatJson·DispatchChatReceived 및 STOMP 수신 콜백.

**원본:** [고정 커밋 소스](https://github.com/AllforOne5Class/CtoC_Unity/blob/11d0b66c1eaf7190a659df8faabc7e542285e7f6/Assets/Scripts/NetworkManager.cs)

원본 범위: `Assets/Scripts/NetworkManager.cs:78–89`

```csharp
    private void Update()
    {
        while (TryDequeueJson(out string json))
        {
            HandleChatJson(json);
        }

        while (TryDequeueChat(out ChatMessageData chatMessage))
        {
            DispatchChatReceived(chatMessage);
        }
    }
```

원본 범위: `Assets/Scripts/NetworkManager.cs:775–781`

```csharp
    private void EnqueueJson(string json)
    {
        lock (_queueLock)
        {
            _receivedJsonQueue.Enqueue(json);
        }
    }
```

원본 범위: `Assets/Scripts/NetworkManager.cs:783–796`

```csharp
    private bool TryDequeueJson(out string json)
    {
        lock (_queueLock)
        {
            if (_receivedJsonQueue.Count == 0)
            {
                json = null;
                return false;
            }

            json = _receivedJsonQueue.Dequeue();
            return true;
        }
    }
```

원본 범위: `Assets/Scripts/NetworkManager.cs:798–804`

```csharp
    private void EnqueueChat(ChatMessageData chatMessage)
    {
        lock (_chatQueueLock)
        {
            _receivedChatQueue.Enqueue(chatMessage);
        }
    }
```

원본 범위: `Assets/Scripts/NetworkManager.cs:806–819`

```csharp
    private bool TryDequeueChat(out ChatMessageData chatMessage)
    {
        lock (_chatQueueLock)
        {
            if (_receivedChatQueue.Count == 0)
            {
                chatMessage = null;
                return false;
            }

            chatMessage = _receivedChatQueue.Dequeue();
            return true;
        }
    }
```

[샘플 목록](../README.md#코드-샘플)
