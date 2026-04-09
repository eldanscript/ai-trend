# Global settings.json 설정 상세 설명

## 파일 위치
`~/.claude/settings.json`

## 설정 항목 상세

### env (환경변수)

| 환경변수 | 값 | 설명 | 필수 여부 |
|---------|---|------|---------|
| `CLAUDE_CODE_EXPERIMENTAL_AGENT_TEAMS` | `"1"` | 에이전트 팀 기능을 활성화한다. TeamCreate, SendMessage, TaskCreate 등 팀 도구가 사용 가능해진다. **Harness 사용의 전제 조건.** | **필수** |
| `ANTHROPIC_MODEL` | 모델 ID | Claude Code의 기본 모델을 지정한다. Bedrock 사용 시 `global.anthropic.claude-opus-4-6-v1` 형식. | 권장 |
| `ANTHROPIC_DEFAULT_OPUS_MODEL` | 모델 ID | Opus 모델 ID. Agent 호출 시 `model: "opus"`로 참조된다. | 권장 |
| `ANTHROPIC_DEFAULT_SONNET_MODEL` | 모델 ID | Sonnet 모델 ID. 빠른 작업에 사용. | 선택 |
| `ANTHROPIC_DEFAULT_HAIKU_MODEL` | 모델 ID | Haiku 모델 ID. 경량 작업에 사용. | 선택 |
| `CLAUDE_CODE_USE_BEDROCK` | `"1"` | AWS Bedrock을 통해 Claude API를 사용할 때 설정. | 조건부 |
| `AWS_PROFILE` | 프로필명 | Bedrock 사용 시 AWS SSO 프로필. | 조건부 |
| `AWS_REGION` | 리전 | Bedrock 사용 시 AWS 리전. | 조건부 |

### awsAuthRefresh (선택)

Bedrock 사용 시 AWS SSO 세션이 만료되면 자동으로 실행할 명령어:
```json
"awsAuthRefresh": "aws sso login --profile <프로필명>"
```

### permissions (선택)

글로벌 수준에서 허용할 도구 권한:
```json
"permissions": {
  "allow": [
    "Bash(git:*)",
    "Bash(npm:*)"
  ]
}
```

## 모델 선택 가이드

| 모델 | 용도 | 비용 |
|------|------|------|
| **Opus** | 에이전트 팀, 복잡한 추론, 코드 생성 | 높음 |
| **Sonnet** | 일반 대화, 빠른 코드 수정 | 중간 |
| **Haiku** | 분류, 간단한 질의 | 낮음 |

> **Harness에서는 모든 에이전트에 Opus를 권장합니다.** 에이전트의 추론 능력이 하네스 품질에 직결됩니다.

## Anthropic API 직접 사용 시

Bedrock 대신 Anthropic API를 직접 사용하는 경우:
```json
{
  "env": {
    "CLAUDE_CODE_EXPERIMENTAL_AGENT_TEAMS": "1",
    "ANTHROPIC_API_KEY": "sk-ant-...",
    "ANTHROPIC_MODEL": "claude-opus-4-6-20250612"
  }
}
```
