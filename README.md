# Encar 테스트코드 스터디

## 소개

Encar에서 진행하는 테스트코드 스터디 레포지토리입니다.

테스트코드 강의를 듣고, 주요 개념을 요약 및 공유하며 리뷰하는 시간을 갖습니다.

## 강의 소개

- [실무에 바로 적용하는 프런트엔드 테스트 - 1부. 테스트 기초: 단위・통합 테스트](https://inf.run/rVcLN)
- [실무에 바로 적용하는 프런트엔드 테스트 - 2부. 테스트 심화: 시각적 회귀・E2E 테스트](https://inf.run/zwz4W)

## 스터디 진행 방식 📝

- **주 하나의 섹션**을 듣고 각자 정리합니다.(기초 내용인 섹션1, 섹션2 정리는 선택 사항)
- 요약 내용은 각 섹션 폴더에 Markdown 형식으로 "[영문이름].md" 커밋합니다.
- **매주 1회** 만나서 챕터별 정리한 내용을 토대로 리뷰하고, 서로의 의견을 공유합니다.
- 스터디 전날 자정까지 정리 내용 푸시하기, 미이행 시 벌금 6,000원 발생

## 스터디 진행 일정

1. 2월 18일(화) - 섹션 3 단위 테스트란? & 섹션 4 단위 테스트 작성하기(55분)
2. 2월 25일(화) - 섹션 5 통합 테스트란?(1시간 15분)
3. 3월 04일(화) - 섹션 6 통합 테스트 작성하기(58분)
4. 3월 10일(월) - 섹션 4 스토리북과 크로마틱을 활용한 시각적 회귀 테스트(51분)
5. 3월 18일(월) - 섹션 5 E2E 테스트란?(47분)
6. 3월 25일(월) - 섹션 6 Cypress를 활용한 E2E 테스트 작성하기 & 섹션 7 마무리(1시간 36분)

## 커밋 규칙

"n부\_n - 깃헙아이디" 규칙으로 올린다.

(ex. 1부\_1 - youngrove)

## vitest 사용하는 법

- 현재 강의 기준으로 vscode 익스텐션인 vitest가 버전 문제로 정상적으로 동작하지 않는 이슈가 있음.
- 해결 방법
  - "vitest": "^1.6.0"
  - "@vitest/ui": "^1.6.0",
  - vitest extenstion 0.10.7
