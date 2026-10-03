# 팀원 크레인 이동 코드에 채팅 명령 연결

방향 이벤트를 정해진 이동 거리로 변환하고 월드 X/Z 경계로 제한합니다. Rigidbody가 있으면 MovePosition, 없으면 Transform을 사용합니다. 이벤트 구독과 입력 잠금은 기존 클래스의 다른 메서드에 있습니다.

**본인 역할:** 팀원이 구현한 MotorMover에 방향 이벤트 처리·거리 이동·경계 제한을 추가했습니다. 아래는 본인이 추가한 로직의 발췌입니다.

**형식:** 일부 발췌. 독립 실행 파일이 아니며 클래스 필드와 나머지 메서드를 포함하지 않습니다.

**의존성:** Unity Rigidbody/Transform/Vector2/Vector3/Mathf, ChatDirection·CraneEventHub, rb·거리·경계·잠금 필드, Stop 및 GetMoveInput.

**원본 파일:** `Assets/Scripts/MotorMover.cs`

원본 범위: `Assets/Scripts/MotorMover.cs:118–124`

```csharp
    private void HandleDirectionCommandRequested(ChatDirection direction)
    {
        if (isInputLocked)
            return;

        MoveByChatDistance(GetMoveInput(direction));
    }
```

원본 범위: `Assets/Scripts/MotorMover.cs:194–215`

```csharp
    private void MoveByChatDistance(Vector2 direction)
    {
        Vector2 normalizedDirection = direction.normalized;
        if (normalizedDirection == Vector2.zero)
        {
            Stop();
            return;
        }

        Stop();

        Vector3 currentPosition = rb != null ? rb.position : transform.position;
        Vector3 offset = new Vector3(normalizedDirection.x, 0f, normalizedDirection.y) * chatMoveDistance;
        Vector3 targetPosition = ClampWorldXZ(currentPosition + offset);

        if (rb != null)
            rb.MovePosition(targetPosition);
        else
            transform.position = targetPosition;

        Stop();
    }
```

원본 범위: `Assets/Scripts/MotorMover.cs:240–247`

```csharp
    private Vector3 ClampWorldXZ(Vector3 position)
    {
        return new Vector3(
            Mathf.Clamp(position.x, minX, maxX),
            position.y,
            Mathf.Clamp(position.z, minZ, maxZ)
        );
    }
```

[샘플 목록](../README.md#코드-샘플)
