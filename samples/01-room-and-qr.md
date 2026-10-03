# 방 생성 결과를 STOMP 구독과 QR 표시로 연결

REST 방 생성이 끝난 뒤 방 ID로 구독 주소를 구성합니다. 방 생성 응답의 data가 문자열이면 방 ID로, 객체이면 방 ID·QR URL 필드로 해석하고 QR URL이 없으면 대체 URL을 사용합니다. 구독 실패는 주소별로 처리합니다.

**본인 역할:** 클라이언트 연결·응답 처리 구현. 백엔드 방 생성 API와 웹 참여 페이지는 팀 구성요소입니다.

**형식:** 일부 발췌. 독립 실행 파일이 아니며 클래스 필드와 나머지 메서드를 포함하지 않습니다.

**의존성:** Unity 6000.5.0f1, UnityWebRequest/uGUI, Netina.Stomp.Client, Newtonsoft.Json, CancellationToken/Task; CreateRoomAsync·RoomCreateData·구독 주소 해석·QR 로딩 메서드와 비공개 연결 설정.

**원본:** [고정 커밋 소스](https://github.com/AllforOne5Class/CtoC_Unity/blob/11d0b66c1eaf7190a659df8faabc7e542285e7f6/Assets/Scripts/NetworkManager.cs)

원본 범위: `Assets/Scripts/NetworkManager.cs:124–164`

```csharp
    private async Task ConnectAndSubscribeAsync(CancellationToken cancellationToken)
    {
        try
        {
            cancellationToken.ThrowIfCancellationRequested();

            RoomCreateData roomData = await CreateRoomAsync(cancellationToken);
            string roomId = roomData.roomId;
            Debug.Log($"Chat room created: {roomId}");
            LoadQrImageIfAvailable(roomData);

            List<string> resolvedSubscribeDestinations = GetSubscribeDestinations(roomId);
            Debug.Log($"Chat STOMP connecting: url={stompUrl}, destinations={string.Join(", ", resolvedSubscribeDestinations)}");

            _stompClient = await ConnectStompAsync(stompUrl, fallbackStompUrl, cancellationToken);

            for (int i = 0; i < resolvedSubscribeDestinations.Count; i++)
            {
                string destination = resolvedSubscribeDestinations[i];
                try
                {
                    await _stompClient.SubscribeAsync(destination, new Dictionary<string, string>(), HandleStompMessage);
                    Debug.Log($"Chat STOMP subscribed: {destination}");
                }
                catch (Exception exception)
                {
                    Debug.LogWarning($"Chat STOMP subscribe failed: destination={destination}, error={exception.Message}");
                }
            }

            Debug.Log($"Chat STOMP connected: {_connectedStompUrl}, subscribedCount={resolvedSubscribeDestinations.Count}");
        }
        catch (OperationCanceledException)
        {
            // Expected when the scene unloads or the object is destroyed.
        }
        catch (Exception exception)
        {
            Debug.LogError($"Chat server setup error: {exception.Message}");
        }
    }
```

원본 범위: `Assets/Scripts/NetworkManager.cs:211–252`

```csharp
    private RoomCreateData ParseRoomCreateData(string responseJson)
    {
        if (string.IsNullOrWhiteSpace(responseJson))
        {
            return null;
        }

        JToken rootToken = JToken.Parse(responseJson);
        JToken dataToken = rootToken["data"];
        if (dataToken == null || dataToken.Type == JTokenType.Null)
        {
            return null;
        }

        if (dataToken.Type == JTokenType.String)
        {
            var roomData = new RoomCreateData
            {
                roomId = dataToken.Value<string>()
            };
            roomData.qrImageUrl = ResolveFallbackQrImageUrl(roomData.roomId);
            return roomData;
        }

        if (dataToken.Type != JTokenType.Object)
        {
            return null;
        }

        var dataObject = (JObject)dataToken;
        var parsedRoomData = new RoomCreateData
        {
            roomId = GetStringValue(dataObject, "roomId", "id"),
            qrImageUrl = GetStringValue(dataObject, "qrImageUrl", "qrUrl", "qrCodeUrl", "qrCodeImageUrl", "qrImage")
        };
        if (string.IsNullOrWhiteSpace(parsedRoomData.qrImageUrl))
        {
            parsedRoomData.qrImageUrl = ResolveFallbackQrImageUrl(parsedRoomData.roomId);
        }

        return parsedRoomData;
    }
```

[샘플 목록](../README.md#코드-샘플)
