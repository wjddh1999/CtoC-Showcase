# 채팅 카드 적용과 선택적 이미지 캐시

닉네임·메시지를 표시하고 이전 이미지 코루틴을 중단한 뒤 새 프로필을 처리합니다. URL 캐시는 옵션이며 최종 채팅 프리팹 저장값은 꺼짐입니다.

**본인 역할:** 채팅 카드 및 프로필 표시 구현.

**형식:** 일부 발췌. 독립 실행 파일이 아니며 클래스 필드와 나머지 메서드를 포함하지 않습니다.

**의존성:** Unity/uGUI, TMPro, ChatMessageData, ProfileSpriteCache·프로필 필드, 이미지 다운로드·캐시 해제·자식 참조 조회·문자 치환 메서드.

**원본 파일:** `Assets/Scripts/ChatItemView.cs`

원본 범위: `Assets/Scripts/ChatItemView.cs:21–70`

```csharp
    internal void Apply(ChatMessageData data)
    {
        if (data == null)
        {
            return;
        }

        ResolveReferencesIfNeeded();

        if (chatText != null)
        {
            chatText.text = $"{data.nickname}: {GetDisplayMessage(data.message)}";
        }

        if (_profileImageRoutine != null)
        {
            StopCoroutine(_profileImageRoutine);
            _profileImageRoutine = null;
        }

        if (profileImage == null)
        {
            return;
        }

        profileImage.preserveAspect = true;
        profileImage.sprite = null;

        if (string.IsNullOrWhiteSpace(data.profileImageUrl))
        {
            //Debug.Log("Profile image url is empty. Profile sprite cleared.");
            return;
        }

        if (useProfileImageCache)
        {
            RegisterCacheReleaseIfNeeded();

            if (TryApplyCachedProfileImage(data.profileImageUrl))
            {
                return;
            }
        }
        else
        {
            //Debug.Log($"Profile image cache disabled: url={data.profileImageUrl}");
        }

        _profileImageRoutine = StartCoroutine(LoadProfileImage(data.profileImageUrl));
    }
```

원본 범위: `Assets/Scripts/ChatItemView.cs:110–122`

```csharp
    private bool TryApplyCachedProfileImage(string imageUrl)
    {
        if (!ProfileSpriteCache.TryGetValue(imageUrl, out Sprite sprite) || sprite == null)
        {
            ProfileSpriteCache.Remove(imageUrl);
            return false;
        }

        Debug.Log($"Profile image cache hit: url={imageUrl}");
        profileImage.preserveAspect = true;
        profileImage.sprite = sprite;
        return true;
    }
```

[샘플 목록](../README.md#코드-샘플)
