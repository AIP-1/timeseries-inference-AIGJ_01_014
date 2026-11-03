# 시계열 데이터 추론 실습 (timeseries-inference)

정원일 · AIGJ_01_014 · Jetson Orin Nano Super

교육장 센서(CO₂·온도·습도)와 냉난방 데이터로 "지금 무슨 일이 일어나고 있고, 곧 어떻게 될까"에 답하는 추론 기능을 만들어 Jetson에서 상시 운영한다. 과제 안내서 원문은 [`docs/assignment.md`](docs/assignment.md).

## 최종 제출물 (안내서 4절)

| 제출물 | 문서 |
| --- | --- |
| 결과 보고서 (과제 1~8) | [`docs/submission/01-final-report.md`](docs/submission/01-final-report.md) |
| 상시 서비스 (수집기·추론 서비스, 조회 API) | [`docs/submission/02-service.md`](docs/submission/02-service.md) |
| 결과 형식 문서 | [`docs/submission/03-result-format.md`](docs/submission/03-result-format.md) |
| 발표 (5분) | [`docs/submission/04-presentation.md`](docs/submission/04-presentation.md) |
| 안내서 조건 대비 수행 점검 | [`docs/submission/00-readme-audit.md`](docs/submission/00-readme-audit.md) |

## 과제별 위치

| 과제 | 산출물 | 점검 문서 | 코드 | 실행 |
| --- | --- | --- | --- | --- |
| 1. 데이터 수집 체계 | [`docs/deliverables/task1`](docs/deliverables/task1) | [task1](docs/tasks/task1-data-collection.md) | [`src/collector`](src/collector) | `tools/collector` |
| 2. 데이터 이해와 기준선 | [`docs/deliverables/task2`](docs/deliverables/task2) | [task2](docs/tasks/task2-data-baseline.md) | [`src/analysis`](src/analysis) | `tools/analysis` |
| 3. 60분 예측 | [`docs/deliverables/task3`](docs/deliverables/task3) | [task3](docs/tasks/task3-forecast-60min.md) | [`src/forecast`](src/forecast) | `tools/forecast` |
| 4. 이상 탐지 | [`docs/deliverables/task4`](docs/deliverables/task4) | [task4](docs/tasks/task4-anomaly-detection.md) | [`src/anomaly`](src/anomaly) | `tools/anomaly` |
| 5. 상태 추정 | [`docs/deliverables/task5`](docs/deliverables/task5) | [task5](docs/tasks/task5-state-estimation.md) | [`src/state`](src/state) | `tools/state` |
| 6. 사전학습 모델 비교 | [`docs/deliverables/task6`](docs/deliverables/task6) | [task6](docs/tasks/task6-pretrained-models.md) | [`src/pretrained`](src/pretrained) | `tools/pretrained` |
| 7. 엣지 최적화 | [`docs/deliverables/task7`](docs/deliverables/task7) | [task7](docs/tasks/task7-edge-optimization.md) | [`src/edgeopt`](src/edgeopt) | `tools/edgeopt` |
| 8. 상시 추론 서비스 | [`docs/deliverables/task8`](docs/deliverables/task8) | [task8](docs/tasks/task8-inference-service.md) | [`src/service`](src/service), [`src/preprocess`](src/preprocess) | `tools/infer` |

- 심화(B) 결과: [`docs/advanced/taskN/`](docs/advanced) · 과제별 계획(spec): [`specs/00N-*/`](specs) · 테스트: [`tests/<과제 패키지>/`](tests)
- 설정: [`config/<과제>.yaml`](config) · 상시 서비스: [`deploy/`](deploy) · 운영 가이드: [`docs/ops/collector-ops.md`](docs/ops/collector-ops.md)

## 실행

저장소를 받은 폴더에서 그대로 실행한다 (경로는 모두 저장소 기준 상대 경로).

```bash
cp .env.example .env                                   # MQTT·MySQL·InfluxDB 접속 정보 입력 (커밋하지 않음)
~/.local/bin/uv venv .venv && ~/.local/bin/uv pip install --python .venv/bin/python -e '.[test,analysis]'
.venv/bin/python -m pytest                             # 단위 + 시험 브로커 (-m mysql / -m live 는 실서버)

tools/collector check                                  # 과제 1: 연결 점검 → deploy/install.sh 로 상시 수집
tools/analysis all                                     # 과제 2: MySQL → 정제 → 분할 → 기준선 → 진단서
deploy/install-inference.sh && tools/infer status      # 과제 8: 추론 서비스 설치·상태
curl -s "http://<보드 주소>:18080/v1/spaces/meeting/forecast?minutes=30"
```

과제별 상세 명령은 각 점검 문서(`docs/tasks/`)의 "확인 명령"에 있다.

환경: Jetson (L4T R39, aarch64, RAM 7.3GB), Python 3.12, sudo·Docker 없이 systemd 사용자 서비스(linger)로 상시 실행. 수집 데이터(MySQL)와 학습된 모델 묶음(`var/`)은 저장소에 없다.
