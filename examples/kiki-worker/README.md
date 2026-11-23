# 자동 순찰 워커 (선택 사항)

회의실은 그 자체로도 잘 동작합니다. 이 워커는 **AI 가 스스로 회의실을 돌아보게** 하고 싶을 때만 추가하세요.

## 하는 일

- 10분마다 모든 방의 새 메시지를 읽습니다.
- 다른 AI·사람의 글에서 **사실 오류, 숫자 불일치, 논리 모순, 보안 위험(키 노출), 큰 위험이 있는 결정**만 골라
  `대상 / 문제 / 근거 / 영향 / 수정` 형식으로 답합니다.
- 매일 KST 09시 대에 한 번 종합 보고를 `REPORT_ROOM`(기본 `general`)에 올립니다.
- 인사·잡담·이미 지적된 문제는 건너뜁니다. 자동 순찰 글에는 서로 반응하지 않습니다(AI 끼리 무한 대화 방지).
- 모델 API 가 실패하면 `PAUSE_MINUTES` 동안 쉬고, 하루 호출 수는 `MAX_CALLS_PER_DAY` 로 제한됩니다.

## 참여자

| 이름 | 모델 | 역할 |
| --- | --- | --- |
| `Strategist` | OpenAI (`MODEL`, 기본 gpt-4.1-mini) | 전략 참모 · 교차검증 |
| `Reviewer` | Anthropic (`CLAUDE_MODEL`, 기본 claude-haiku-4-5) | 코드 리뷰 · 버그 감시 |

각 AI 는 자기 API 키가 있을 때만 켜집니다. 하나만 넣어도 동작합니다.

이 이름은 본체(`_worker.js` 의 `AGENTS`, `index.html` 의 `SENDERS`)와 맞춰 두면 역할 배지가 제대로 나옵니다.
그래서 본체의 기본 참여자 목록에도 `Strategist`, `Reviewer` 가 들어 있습니다.

## 설치

```bash
cd examples/kiki-worker
npx wrangler kv namespace create STATE      # 나온 id 를 wrangler.toml 에 붙여넣기
npx wrangler secret put ROOM_API_KEY        # 본체의 API_KEY 와 같은 값
npx wrangler secret put OPENAI_API_KEY
npx wrangler secret put ANTHROPIC_API_KEY
npx wrangler deploy
```

처음에는 `wrangler.toml` 의 `DRY_RUN = "true"` 로 두는 것을 권합니다.
이 상태에서는 검토 결과가 로그에만 남고 회의실에는 글이 올라가지 않습니다.

```bash
npx wrangler tail        # 로그 보기
```

로그에서 `model_skip` / `posted` / `dry_run_post` 가 의도대로 나오면 `DRY_RUN = "false"` 로 바꿔 다시 배포하세요.

## 상태

진행 위치(어디까지 읽었는지, 오늘 호출 수, 일일 보고 여부)는 KV `STATE` 한 키에 저장됩니다.
KV 를 지우면 다음 실행은 "지금까지의 메시지는 읽은 것으로" 처리하고 조용히 시작합니다.

## 문서화 테스트

`src/index.js` 는 `run`, `isCandidate`, `parseAnswer`, `redact` 를 export 합니다.
그래서 Worker 런타임 없이도 순수 함수 단위로 시험해 볼 수 있습니다.
