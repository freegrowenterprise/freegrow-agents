---
description: 개인 하네스를 역할별/공통으로 승격 (PR 생성)
---

# Harness Promote

개인 하네스를 역할별 또는 공통 하네스로 승격합니다. freegrow-harness 플러그인 레포에 PR을 자동 생성합니다.

## 인자

`$ARGUMENTS` - 승격할 하네스 ID (예: `backend-001`)

## 전제 조건

- `~/.claude/harness/personal/` 디렉토리가 존재해야 함
- 해당 ID의 하네스 파일이 `~/.claude/harness/personal/rules/`에 존재해야 함

## 실행 절차

### 1. 하네스 파일 읽기

`$ARGUMENTS`에서 하네스 ID를 추출하고, `~/.claude/harness/personal/rules/{id}.md` 파일을 읽습니다.

### 2. 승격 대상 확인

```
🚀 하네스 승격

ID: {id}
내용: "{규칙 내용}"
현재 역할: {role}

어디로 승격할까요?
1. {role}별 공통 → roles/{role}.md에 추가
2. 전사 공통 → CLAUDE.md에 추가
```

### 3. PR 생성 안내

Claude Code 플러그인은 별도의 Git 레포로 관리되므로, 승격은 직접 PR을 생성하는 방식으로 진행합니다.

```
📋 승격 내용을 준비했습니다.

아래 규칙을 freegrow-harness 플러그인 레포의 {대상 파일}에 추가하는 PR을 생성해주세요:

---
{규칙 내용}
---

레포: freegrowenterprise/freegrow-agents
대상 파일: plugins/freegrow-harness/{대상 파일 경로}

gh를 사용하여 PR을 생성할까요? (y/n)
```

### 4. PR 자동 생성 (사용자 승인 시)

```bash
# freegrow-agents 레포 경로에서 실행
cd {freegrow-agents 레포 경로}
git checkout -b harness/promote-{id}
# 대상 파일에 규칙 추가
git add {대상 파일}
git commit -m "feat: 하네스 승격 - {id} ({규칙 요약})"
git push origin harness/promote-{id}
gh pr create --title "[FEAT] 하네스 승격: {규칙 요약}" --body "## 📋 Summary
개인 하네스를 {대상} 하네스로 승격합니다.

## 하네스 내용
{규칙 내용}

## 출처
- 원본 ID: {id}
- 감지 방식: {source}
- 트리거 횟수: {trigger_count}회" --assignee @me
```

### 5. 결과 출력

```
✅ PR 생성 완료!

PR: #{PR번호} [FEAT] 하네스 승격: {규칙 요약}
URL: {PR URL}

팀 리뷰 후 머지되면 모두에게 적용됩니다.
```
