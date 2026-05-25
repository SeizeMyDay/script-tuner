# 프로젝트 진행 현황

> 본 문서는 프로젝트 상태 추적용이다. 정적 설계는 `docs/design/`, 결정 이력은 `docs/decisions/`(ADR), 개인 작업 메모는 `.work/`(gitignore)에 둔다.

마지막 업데이트: 2026-05-25

---

## 마일스톤

**M0~M12: PoC 전처리 파이프라인 완료** ✅

SBC016 한 파일에 대해 `scripttuner run sbcsae SBC016` (parse -> clean -> monologue -> pairs -> stats) 단일 명령으로 end-to-end 검증 완료. 자세한 진행 이력은 `git log` 및 아래 ADR 목록 참조.

| # | 마일스톤 | 상태 | 비고 |
|---|---|---|---|
| M13 | SBCSAE 60개 확장 적용 | ✅ 완료 | 60개 SBCSAE 파일 배치 전처리 완료. API 기반 LLM 변환 결과를 프로젝트 구조에 맞게 `data/pairs/SBCSAE/_all.jsonl`로 배치했고, 집계 통계는 `data/stats/SBCSAE/_aggregate.json`에 추가. |
| 이후 | 진단 모듈 / 변환 모델 / 백엔드 / UI | 미정 | 본격 학습 및 서비스화 단계에서 구체화 |

## 2026-05-25 업데이트

- **M13 완료 범위**: SBCSAE 60개 파일 확장 전처리 완료. `data/training_ready_pre_llm/SBCSAE` 기준 총 monologue 수는 1,757개.
- **LLM 변환 산출물 반영**: 팀원이 API를 통해 생성한 post-LLM Pair 데이터 `data_sbc/_all.jsonl`을 프로젝트 구조에 맞게 `data/pairs/SBCSAE/_all.jsonl`로 추가.
- **집계 통계 반영**: `data_sbc/_aggregate.json`을 `data/stats/SBCSAE/_aggregate.json`로 추가. pair 수는 1,757개, unique speaker 수는 131명.
- **산출물 설명 추가**: `data/pairs/SBCSAE/README.txt`, `data/pairs/SBCSAE/MANIFEST.json` 추가.
- **누락되기 쉬운 진행 사항**: 단순히 60개 원천 파일 확장만 끝난 것이 아니라, API 변환을 거친 최종 Pair aggregate까지 `script-tuner-main`의 `data/pairs` / `data/stats` 구조에 반영된 상태다.

## 결정 이력 (ADR)

- [ADR-0001](decisions/0001-jsonl-output-format.md) — 학습 데이터 출력 포맷으로 JSONL 채택
- [ADR-0002](decisions/0002-sbcsae-license-policy.md) — SBCSAE 라이선스 대응 (다운로드 스크립트 + gitignore)
- [ADR-0003](decisions/0003-pause-marker-tokenization.md) — pause marker 특수 토큰화
- [ADR-0004](decisions/0004-backchannel-handling.md) — backchannel 처리 정책
- [ADR-0005](decisions/0005-style-as-dataset-metadata.md) — style label을 dataset metadata로 관리
- [ADR-0006](decisions/0006-adapter-structure-and-common-ir.md) — adapter 구조 + common IR
- [ADR-0007](decisions/0007-llm-client-provider-agnostic-and-caching.md) — LLM client provider-agnostic + disk caching
- [ADR-0008](decisions/0008-pause-token-strip-on-llm-input.md) — LLM 입력 시 pause token strip, spoken text에는 보존

## 다음 액션 (단기)

1. **학습 데이터셋 구성** — `data/pairs/SBCSAE/_all.jsonl`을 기준으로 train/validation/test 분리. 같은 `speaker` 또는 `monologue_id`가 서로 다른 split에 섞이지 않도록 분리 기준 결정.
2. **변환 방향 확정** — 프로젝트 목표가 OPIc 스크립트의 구어체화라면 학습 방향은 `formal_text -> spoken_text`가 자연스럽다.
3. **본격 학습/모델 단계로 전환** — 베이스 모델 선택, LoRA/QLoRA 학습 파이프라인 구성, 학습용 instruction 포맷 정의.
4. **평가 체계 구체화** — 일반 LLM 프롬프트 결과를 baseline으로 만들고, 자연스러움/의미 보존/OPIc 적합성 평가 기준 정의.

## 보류 / 추후 결정

- **Semi-formal style 데이터 확보 방안** — 인터뷰, TED 등 monologue corpus 후보 조사 필요
- **제어 토큰 학습 전략** — semi-formal 데이터 확보 후 결정
- **진단 모듈 feature set 최종화** — stats 결과를 보고 결정
- **모델 변환 베이스 선택** — T5Gemma 2 vs Gemma 4 등 후보 비교 필요
- **few-shot 도입 시점** — 현재 zero-shot 결과가 양호하므로, 추가 corpus/평가 이슈를 보고 결정
- **정량 평가 metric** — 본격 학습 단계 진입 후 BLEU, embedding similarity 등 도입 결정

## 2026-05-25 update: fine-tuning preparation

- Added fine-tuning preparation modules under `scripttuner/training/`.
- Added style-control definitions for `<STYLE=casual>` and `<STYLE=semi_formal>`.
- Added CLI commands:
  - `scripttuner split sbcsae`
  - `scripttuner format <model_key> sbcsae`
- Generated speaker-aware splits from `data/pairs/SBCSAE/_all.jsonl`:
  - train: 1,405 pairs / 107 speakers
  - validation: 176 pairs / 12 speakers
  - test: 176 pairs / 12 speakers
- Generated model-specific fine-tuning data:
  - `data/finetune/SBCSAE/formatted/gemma4-e4b`
  - `data/finetune/SBCSAE/formatted/gemma4-e2b`
  - `data/finetune/SBCSAE/formatted/qwen3-4b`
  - `data/finetune/SBCSAE/formatted/qwen3-1.7b`
  - `data/finetune/SBCSAE/formatted/t5gemma2`
- Current formatted data contains only `casual` examples. `semi_formal` is reserved for a future external corpus or teacher-LLM generated dataset.
- Added `docs/design/finetuning_pipeline.md` for the fine-tuning preparation flow.
- Verification completed with Python compile checks and actual CLI split/format runs. `pytest` and `ruff` were not available in the local `.venv`.

## 2026-05-25 업데이트: 파인튜닝 사전 준비 상세 정리

### 작업 목적

기존 프로젝트는 SBCSAE 원본 대화 데이터를 `parse -> clean -> monologue -> pairs -> stats` 순서로 처리하여, 최종적으로 `formal_text -> spoken_text` 형태의 병렬 학습 데이터를 만드는 데까지 완료되어 있었다. 이번 업데이트에서는 이 데이터를 실제 경량 언어모델 파인튜닝에 바로 사용할 수 있도록, 학습 데이터 분리와 모델별 포맷 변환 단계를 프로젝트 파이프라인에 추가했다.

### 현재 사용 가능한 학습 원천 데이터

- 원천 파일: `data/pairs/SBCSAE/_all.jsonl`
- 데이터 스키마: `Pair`
- 전체 pair 수: 1,757개
- 고유 speaker 수: 131명
- 학습 방향: `formal_text -> spoken_text`
- 현재 스타일 라벨: `casual`

이 방향은 사용자가 작성한 문어체 또는 부자연스러운 영어 스크립트를 입력하면, 모델이 자연스러운 구어체 영어 답변으로 변환하도록 학습시키는 것을 의미한다. 즉 OPIc 대비용 스크립트 변환 목표와 직접 연결된다.

### 추가된 코드 구조

파인튜닝 준비 코드는 `scripttuner/training/` 아래에 새로 분리했다.

- `scripttuner/training/style.py`
  - `<STYLE=casual>` 및 `<STYLE=semi_formal>` 제어 토큰 정의
  - 각 스타일의 instruction과 설명을 한 곳에서 관리
- `scripttuner/training/split.py`
  - speaker-aware train/validation/test 분리
  - 같은 speaker가 여러 split에 섞이지 않도록 하여 평가 누수를 줄임
- `scripttuner/training/formatters.py`
  - Gemma 4, Qwen 계열용 chat-format 데이터 생성
  - T5Gemma 2용 seq2seq-format 데이터 생성
- `scripttuner/cli.py`
  - `split` 및 `format` 서브커맨드 추가

### 추가된 CLI 단계

새로 추가된 명령은 다음과 같다.

```bash
python -m scripttuner.cli split sbcsae --data-dir data --seed 42
python -m scripttuner.cli format gemma4-e4b sbcsae --data-dir data
python -m scripttuner.cli format gemma4-e2b sbcsae --data-dir data
python -m scripttuner.cli format qwen3-4b sbcsae --data-dir data
python -m scripttuner.cli format qwen3-1.7b sbcsae --data-dir data
python -m scripttuner.cli format t5gemma2 sbcsae --data-dir data
```

### 생성된 split 결과

`data/finetune/SBCSAE/splits/` 아래에 학습, 검증, 테스트 데이터가 생성되었다.

- train: 1,405 pairs / 107 speakers
- validation: 176 pairs / 12 speakers
- test: 176 pairs / 12 speakers

분리는 단순 랜덤이 아니라 speaker 기준으로 수행했다. 이 방식은 같은 화자의 말투, 반복 표현, 전사 특성이 학습 데이터와 테스트 데이터에 동시에 들어가는 것을 줄여서, 모델 성능을 더 정직하게 평가할 수 있게 한다.

### 생성된 모델별 formatted 데이터

`data/finetune/SBCSAE/formatted/` 아래에 모델별 학습 포맷을 생성했다.

- `gemma4-e4b`
  - 1차 주력 모델 후보
  - chat-style fine-tuning 형식
- `gemma4-e2b`
  - 더 가벼운 Gemma 4 비교 모델
  - chat-style fine-tuning 형식
- `qwen3-4b`
  - Gemma 4 E4B와 비교할 강한 대안 모델
  - chat-style fine-tuning 형식
- `qwen3-1.7b`
  - 경량 비교 모델
  - chat-style fine-tuning 형식
- `t5gemma2`
  - encoder-decoder 계열 비교 모델
  - `input` / `target` seq2seq 형식

### 스타일 제어 토큰 계획

제안서의 Casual / Semi-formal spoken 선택 기능을 반영하기 위해, 학습 입력 앞에 스타일 제어 토큰을 붙이는 구조를 도입했다.

```text
<STYLE=casual>
<STYLE=semi_formal>
```

현재 SBCSAE 기반 데이터는 전부 casual spoken 성격이므로, 실제 생성된 학습 데이터에는 `casual` 예시만 포함되어 있다. `semi_formal`은 아직 학습 데이터가 없기 때문에, 토큰과 포맷 구조만 미리 예약해 둔 상태다. 이후 외부 semi-formal corpus 또는 teacher LLM을 사용해 semi-formal target을 생성하면 같은 파이프라인에 추가할 수 있다.

### 현재 남아 있는 주의점

- 현재 `spoken_text`는 pause token과 일부 전사 표기를 보존한다.
- 일부 샘플에는 `[% laugh]` 같은 전사 잔여 표기가 남아 있을 수 있다.
- 최종 모델이 실제 사용자에게 출력할 문장에는 이런 표기가 불필요할 가능성이 높으므로, 본격 학습 전 target cleaning 또는 quality filtering 정책을 추가로 결정해야 한다.
- `pytest`, `ruff`는 현재 로컬 `.venv`에 설치되어 있지 않아 실행하지 못했고, 대신 Python compile check와 실제 CLI 실행으로 검증했다.

### 다음 단계

1. Gemma 4 E4B용 QLoRA/SFT 학습 스크립트 추가
2. 학습 run 산출물 저장 구조 정의: `runs/finetune/`, `runs/eval/`
3. 생성 결과 평가 스크립트 추가
4. target cleaning 정책 결정
5. semi-formal 데이터 확보 방식 결정
6. Gemma 4 E2B, Qwen 계열, T5Gemma 2 비교 실험으로 확장
