# Upstream Fuwari와 동기화하기

이 저장소는 [saicaca/fuwari](https://github.com/saicaca/fuwari)의 포크입니다. 이 문서는 upstream 저장소와 포크를 동기화하는 방법을 설명합니다.

## 문제 해결됨

이제 다음이 수정되었습니다:
- ✅ 저장소가 unshallow되어 전체 git 히스토리를 가지고 있습니다
- ✅ origin의 fetch refspec이 모든 브랜치를 가져오도록 수정되었습니다
- ✅ upstream 리모트가 saicaca/fuwari로 추가되었습니다
- ✅ upstream 연결이 검증되었습니다

## 설정

저장소는 두 개의 리모트로 구성되어 있습니다:
- `origin`: 당신의 포크 (rosmontisu/Blog)
- `upstream`: 원본 저장소 (saicaca/fuwari)

## Upstream과 동기화하는 방법

### 주의사항: 관련 없는 히스토리

이 Blog 저장소는 upstream fuwari 저장소와 별도의 git 히스토리를 가지고 있습니다. 처음 동기화할 때는 `--allow-unrelated-histories` 플래그를 **반드시** 사용해야 합니다.

### 추천 방법: Merge를 사용한 동기화

```bash
# 1. upstream에서 최신 변경사항 가져오기
git fetch upstream

# 2. main 브랜치로 전환
git checkout main

# 3. upstream 변경사항을 main 브랜치에 병합
# 주의: 처음 동기화하거나 "refusing to merge unrelated histories" 오류가 발생하면
# --allow-unrelated-histories 플래그를 사용해야 합니다
git merge upstream/main --allow-unrelated-histories

# 4. 충돌 해결 (많은 충돌이 발생할 수 있습니다)
# 충돌이 발생한 파일을 편집한 후 git add로 추가

# 5. 병합 완료
git merge --continue

# 6. 변경사항을 포크에 푸시
git push origin main
```

### 대안: Pull Request를 통한 안전한 동기화 (추천)

```bash
# 1. main에서 새 브랜치 생성
git checkout main
git checkout -b sync-upstream

# 2. upstream과 병합 (필요시 --allow-unrelated-histories 사용)
git merge upstream/main --allow-unrelated-histories

# 3. 충돌 해결
# - 각 충돌을 신중하게 검토
# - 커스텀 설정은 유지 (src/config.ts 등)
# - upstream의 새 기능과 버그 수정은 받아들임
# - 로컬에서 테스트

# 4. 브랜치 푸시 및 GitHub에서 PR 생성
git push origin sync-upstream
```

## 이 저장소를 위한 권장 동기화 전략

이 저장소는 upstream fuwari와 관련 없는 히스토리를 가지고 있으므로, 다음 접근 방식을 권장합니다:

### 첫 번째 주요 동기화:

1. **Upstream 변경사항 먼저 검토:** upstream/main의 새로운 내용 확인
2. **특정 기능만 선택:** 모든 것을 병합하는 대신 원하는 특정 커밋만 cherry-pick
3. **수동 업데이트:** 주요 기능은 광범위한 충돌을 피하기 위해 수동으로 구현 고려

### 지속적인 동기화:

`--allow-unrelated-histories`로 초기 동기화를 완료하면, 향후 동기화가 더 쉬워집니다:

1. 정기적으로 upstream 가져오기: `git fetch upstream`
2. 동기화 브랜치 생성: `git checkout -b sync-upstream-YYYYMMDD`
3. 점진적으로 병합: 더 자주 동기화하여 충돌 줄이기
4. 철저한 테스트: 푸시하기 전에 항상 로컬에서 테스트

## Upstream의 업데이트 확인

Upstream의 새로운 내용 확인:

```bash
# Upstream 변경사항 가져오기
git fetch upstream

# main 브랜치와 upstream 비교
git log origin/main..upstream/main --oneline

# 상세한 차이점 보기
git diff origin/main upstream/main
```

## Cherry-pick으로 특정 커밋만 가져오기

모든 변경사항이 아닌 특정 변경사항만 원할 경우:

```bash
# 1. Upstream 변경사항 가져오기
git fetch upstream

# 2. Upstream 커밋 확인
git log upstream/main

# 3. 특정 커밋 cherry-pick (COMMIT_HASH를 실제 해시로 교체)
git cherry-pick COMMIT_HASH

# 4. 변경사항 푸시
git push origin main
```

## 문제 해결

### "refusing to merge unrelated histories" 오류
이 저장소에서는 예상되는 오류입니다. `--allow-unrelated-histories` 플래그를 사용하세요:
```bash
git merge upstream/main --allow-unrelated-histories
```

### "Your branch has diverged"
main과 upstream/main이 서로 다른 커밋을 가지고 있다는 의미입니다. 병합하거나 리베이스해야 합니다.

### 병합 충돌
충돌이 발생하면:
1. 충돌이 발생한 파일 열기
2. 충돌 마커(`<<<<<<<`, `=======`, `>>>>>>>`) 찾기
3. 어떤 변경사항을 유지할지 선택하여 충돌 해결
4. `git add <resolved-files>` 실행
5. `git merge --continue`로 병합 완료

### 푸시 실패
푸시가 실패하면:
- 먼저 변경사항을 pull해야 할 수 있음: `git pull origin main`
- 리베이스 후 force push가 필요할 수 있음: `git push origin main --force-with-lease`

## 참고사항

- Upstream 리모트가 이미 이 저장소에 구성되어 있습니다
- 동기화를 시도하기 전에 항상 upstream에서 가져오기를 수행하세요
- 동기화 전에 백업 브랜치 생성을 고려하세요: `git checkout -b backup-main`
- 배포하기 전에 동기화 후 철저히 테스트하세요

## 추가 정보

자세한 내용은 [UPSTREAM_SYNC.md](./UPSTREAM_SYNC.md) 파일(영문)을 참조하세요.
