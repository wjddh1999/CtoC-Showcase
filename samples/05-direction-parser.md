# 한국어·영어·화살표 방향 입력 정규화

공백과 ! . , 문자를 제거하고 소문자로 정규화한 후 별칭과 정확히 비교합니다. 예를 들어 UP!은 위 방향으로 해석하지만 일반 문장에 포함된 방향어는 명령으로 처리하지 않습니다.

**본인 역할:** 방향 별칭·정규화 구현.

**형식:** 일부 발췌. 독립 실행 파일이 아니며 클래스 필드와 나머지 메서드를 포함하지 않습니다.

**의존성:** System, ChatDirection enum. 발췌 클래스 자체는 완전하지만 원본 ChatModels.cs 전체는 아닙니다.

**원본 파일:** `Assets/Scripts/ChatModels.cs`

원본 범위: `Assets/Scripts/ChatModels.cs:53–147`

```csharp
public static class ChatDirectionText
{
    private const string UpLabel = "위";
    private const string DownLabel = "아래";
    private const string LeftLabel = "좌";
    private const string RightLabel = "우";

    private static readonly string[] UpAliases = { "위", "상", "위로", "up", "u", "↑" };
    private static readonly string[] DownAliases = { "아래", "하", "밑", "down", "d", "↓" };
    private static readonly string[] LeftAliases = { "왼쪽", "좌", "left", "l", "←" };
    private static readonly string[] RightAliases = { "오른쪽", "우", "right", "r", "→" };

    public static bool TryParse(string message, out ChatDirection direction)
    {
        string normalizedMessage = Normalize(message);
        int matchCount = 0;
        direction = default;

        TryMatchDirection(normalizedMessage, UpAliases, ChatDirection.Up, ref direction, ref matchCount);
        TryMatchDirection(normalizedMessage, DownAliases, ChatDirection.Down, ref direction, ref matchCount);
        TryMatchDirection(normalizedMessage, LeftAliases, ChatDirection.Left, ref direction, ref matchCount);
        TryMatchDirection(normalizedMessage, RightAliases, ChatDirection.Right, ref direction, ref matchCount);

        if (matchCount == 1)
        {
            return true;
        }

        direction = default;
        return false;
    }

    public static string GetLabel(ChatDirection direction)
    {
        switch (direction)
        {
            case ChatDirection.Up:
                return UpLabel;
            case ChatDirection.Down:
                return DownLabel;
            case ChatDirection.Left:
                return LeftLabel;
            case ChatDirection.Right:
                return RightLabel;
            default:
                return string.Empty;
        }
    }

    private static void TryMatchDirection(
        string normalizedMessage,
        string[] aliases,
        ChatDirection matchedDirection,
        ref ChatDirection direction,
        ref int matchCount)
    {
        for (int i = 0; i < aliases.Length; i++)
        {
            if (normalizedMessage != aliases[i])
            {
                continue;
            }

            direction = matchedDirection;
            matchCount++;
            return;
        }
    }

    private static string Normalize(string message)
    {
        string trimmedMessage = (message ?? string.Empty).Trim().ToLowerInvariant();
        if (trimmedMessage.Length == 0)
        {
            return string.Empty;
        }

        char[] normalizedCharacters = new char[trimmedMessage.Length];
        int normalizedLength = 0;

        for (int i = 0; i < trimmedMessage.Length; i++)
        {
            char character = trimmedMessage[i];
            if (char.IsWhiteSpace(character) || character == '!' || character == '.' || character == ',')
            {
                continue;
            }

            normalizedCharacters[normalizedLength] = character;
            normalizedLength++;
        }

        return new string(normalizedCharacters, 0, normalizedLength);
    }
}
```

[샘플 목록](../README.md#코드-샘플)
