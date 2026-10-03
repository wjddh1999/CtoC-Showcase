# 방향 명령과 집기 요청·재개

유효한 방향 메시지마다 즉시 크레인 명령을 보내고, 누적 건수가 설정값에 도달하면 집기를 요청하며 투표를 중단합니다. 완료 이벤트 또는 타임아웃으로 투표를 재개합니다. 최종 씬 설정은 20건·10초이고 코드 기본값은 40건입니다. 타임아웃은 투표 수신 복구이며 물리 시퀀스 종료를 보장하지 않습니다.

**본인 역할:** 채팅 투표·이벤트 연동 구현. 실제 크레인 이동·집기의 기반 구현은 팀원 작업이며 후속 연동은 공동 작업입니다.

**형식:** 일부 발췌. 독립 실행 파일이 아니며 클래스 필드와 나머지 메서드를 포함하지 않습니다.

**의존성:** Unity/uGUI/TMP, ChatMessageData·ChatDirectionText, CraneEventHub, CraneGrabber, 투표 필드·그래프 갱신·타임아웃 시작/중단 메서드.

**원본 파일:** `Assets/Scripts/ChatTestController.cs`

원본 범위: `Assets/Scripts/ChatTestController.cs:118–171`

```csharp
    private void CollectDirectionVote(ChatMessageData chatMessage)
    {
        if (!_acceptsDirectionVotes)
        {
            return;
        }

        if (!TryParseDirection(chatMessage.message, out ChatDirection direction))
        {
            return;
        }

        if (!string.IsNullOrEmpty(chatMessage.nickname))
        {
            _candidateNicknames.Add(chatMessage.nickname);
        }

        CraneEventHub.RequestDirectionCommand(direction);

        switch (direction)
        {
            case ChatDirection.Up:
                _upVotes++;
                break;
            case ChatDirection.Down:
                _downVotes++;
                break;
            case ChatDirection.Left:
                _leftVotes++;
                break;
            case ChatDirection.Right:
                _rightVotes++;
                break;
        }

        _totalDirectionVotes++;
        countText.text = $"{_totalDirectionVotes} / {VotesBeforePickupRequest}";

        if (_totalDirectionVotes >= VotesBeforePickupRequest)
        {
            // Vote totals are UI-only and reset when pickup starts.
            ResetDirectionVotes();
            _acceptsDirectionVotes = false;
            StartPickupTimeout();
            if (!CraneEventHub.RequestCranePickup())
            {
                RequestCranePickupFallback();
            }

            return;
        }

        UpdateVoteGraph();
    }
```

원본 범위: `Assets/Scripts/ChatTestController.cs:173–185`

```csharp
    private void RequestCranePickupFallback()
    {
        if (craneGrabber != null && craneGrabber.RequestDrop())
        {
            return;
        }

        Debug.LogWarning("Crane pickup request had no listener and no available CraneGrabber fallback. Direction voting resumed.");
        StopPickupTimeout();
        _candidateNicknames.Clear();
        _acceptsDirectionVotes = true;
        ResetDirectionVotes();
    }
```

원본 범위: `Assets/Scripts/ChatTestController.cs:263–269`

```csharp
    private void HandleCranePickupCompleted()
    {
        StopPickupTimeout();
        _candidateNicknames.Clear();
        _acceptsDirectionVotes = true;
        ResetDirectionVotes();
    }
```

원본 범위: `Assets/Scripts/ChatTestController.cs:294–308`

```csharp
    private IEnumerator PickupTimeoutRoutine()
    {
        yield return new WaitForSeconds(pickupTimeoutSeconds);

        _pickupTimeoutCoroutine = null;
        if (_acceptsDirectionVotes)
        {
            yield break;
        }

        Debug.LogWarning("Crane pickup was not completed before timeout. Direction voting resumed.");
        _candidateNicknames.Clear();
        _acceptsDirectionVotes = true;
        ResetDirectionVotes();
    }
```

[샘플 목록](../README.md#코드-샘플)
