# 채팅 응답 형태와 필드명 정규화

배열·data 래퍼를 재귀적으로 해석하고 message/content/text 등의 필드명을 공통 ChatMessageData로 변환합니다. writer 내부 닉네임·이미지 필드도 확인합니다. 객체에 메시지가 없으면 건너뛰며, 객체 외 토큰은 익명 메시지로 변환합니다. 임의 JSON 스키마 전체를 지원하는 범용 파서는 아닙니다.

**본인 역할:** 유연 JSON 파싱과 채팅 데이터 변환 구현.

**형식:** 일부 발췌. 독립 실행 파일이 아니며 클래스 필드와 나머지 메서드를 포함하지 않습니다.

**의존성:** Newtonsoft.Json.Linq, ChatMessageData, FallbackNickname 상수, EnqueueChat; 호출부 HandleFlexibleChatJson의 예외 처리.

**원본:** [고정 커밋 소스](https://github.com/AllforOne5Class/CtoC_Unity/blob/11d0b66c1eaf7190a659df8faabc7e542285e7f6/Assets/Scripts/NetworkManager.cs)

원본 범위: `Assets/Scripts/NetworkManager.cs:626–669`

```csharp
    private void EnqueueParsedChatToken(JToken token)
    {
        if (token == null || token.Type == JTokenType.Null)
        {
            return;
        }

        if (token.Type == JTokenType.Array)
        {
            foreach (JToken item in token.Children())
            {
                EnqueueParsedChatToken(item);
            }

            return;
        }

        if (token.Type != JTokenType.Object)
        {
            EnqueueChat(new ChatMessageData
            {
                nickname = FallbackNickname,
                message = token.ToString()
            });
            return;
        }

        var chatObject = (JObject)token;
        JToken dataToken = chatObject["data"];
        if (dataToken != null && dataToken.Type != JTokenType.Null)
        {
            EnqueueParsedChatToken(dataToken);
            return;
        }

        ChatMessageData chatMessage = CreateChatMessage(chatObject);
        if (chatMessage == null)
        {
            Debug.LogWarning($"Chat JSON did not include a message field.\n{chatObject}");
            return;
        }

        EnqueueChat(chatMessage);
    }
```

원본 범위: `Assets/Scripts/NetworkManager.cs:671–698`

```csharp
    private ChatMessageData CreateChatMessage(JObject chatObject)
    {
        string message = GetStringValue(chatObject, "message", "content", "text", "chat", "msg", "body");
        if (string.IsNullOrWhiteSpace(message))
        {
            return null;
        }

        JObject writerObject = GetObjectValue(chatObject, "writer");
        string nickname = GetStringValue(chatObject, "nickname", "nickName", "username", "userName", "sender", "name", "displayName");
        if (string.IsNullOrWhiteSpace(nickname))
        {
            nickname = GetStringValue(writerObject, "nickname", "nickName", "username", "userName", "sender", "name", "displayName");
        }

        string profileImageUrl = GetStringValue(chatObject, "profileImageUrl", "profileImage", "profileUrl", "imageUrl", "avatarUrl", "avatar");
        if (string.IsNullOrWhiteSpace(profileImageUrl))
        {
            profileImageUrl = GetStringValue(writerObject, "profileImageUrl", "profileImage", "profileUrl", "imageUrl", "avatarUrl", "avatar");
        }

        return new ChatMessageData
        {
            profileImageUrl = profileImageUrl,
            nickname = string.IsNullOrWhiteSpace(nickname) ? FallbackNickname : nickname,
            message = message
        };
    }
```

원본 범위: `Assets/Scripts/NetworkManager.cs:700–723`

```csharp
    private string GetStringValue(JObject chatObject, params string[] names)
    {
        if (chatObject == null)
        {
            return string.Empty;
        }

        for (int i = 0; i < names.Length; i++)
        {
            JToken token = chatObject[names[i]];
            if (token == null || token.Type == JTokenType.Null)
            {
                continue;
            }

            string value = token.Type == JTokenType.String ? token.Value<string>() : token.ToString();
            if (!string.IsNullOrWhiteSpace(value))
            {
                return value;
            }
        }

        return string.Empty;
    }
```

원본 범위: `Assets/Scripts/NetworkManager.cs:725–742`

```csharp
    private JObject GetObjectValue(JObject chatObject, params string[] names)
    {
        if (chatObject == null)
        {
            return null;
        }

        for (int i = 0; i < names.Length; i++)
        {
            JToken token = chatObject[names[i]];
            if (token is JObject value)
            {
                return value;
            }
        }

        return null;
    }
```

[샘플 목록](../README.md#코드-샘플)
