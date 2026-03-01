| 세트 | 대표 Topic                                          | Target Audience                        | Desired Impact                            |
| -- | ------------------------------------------------- | -------------------------------------- | ----------------------------------------- |
| 1  | 생산성, 습관 설계, 시간관리, 실행 시스템, 성공 루틴                   | 커리어 초기·중반 직장인, 실행력 향상을 원하는 자기계발 독자     | 통제감 회복, 행동 변화 촉진, 성과·수입 상승 가능성 기대 형성      |
| 2  | 마인드셋, 자존감 회복, 인식 전환, 삶의 의미, 감정 해방                 | 번아웃 직장인, 방향성 혼란층, 심리적 위안과 자기확신을 원하는 독자 | 심리적 해방감 제공, 자기확신 강화, 성장 중이라는 체감 형성        |
| 3  | 부 축적 전략, 재정 자유, 수동 소득, 기업가 사고방식, 성공 사례 기반 비즈니스 전략 | 재정 자유 추구 전문직, 예비 창업가, 부업·사업 확장 관심층     | 경제적 독립 욕구 자극, 사업 실행 동기 강화, 빠른 성과 달성 기대 제공 |



Gemini CLI의 에이전트 스킬(Skill) 시스템과 쉘 실행 권한을 결합하면, 말씀하신 가상
  팀(Virtual Team)을 구성하고 복잡한 워크플로우를 자동화하는 것이 충분히 가능합니다.

  이를 실현하기 위한 최적의 워크플로우와 Gemini CLI 설계안을 제안해 드립니다.

  ---

  1. 가상 팀 구성 전략 (Agent Personas)

  Gemini CLI 내부에 각 역할별로 전문화된 스킬(`.agent/skills`)을 등록하여 각 에이전트의 페르소나와 제약 사항을 정의합니다.


   * `agent-leader` (Master Skill): 전체 로드맵(TOC.md) 설계, 각 에이전트에게 할당할 프롬프트 생성, 전체 진행 상황 관리.
   * `agent-writer` (Content Skill): STEM 수준의 엄밀한 수학적 텍스트 생성 (LaTeX 수식 포함).
   * `agent-drawer` (Visual Skill): 기하학적 의미를 담은 SVG/XML 코드 생성 (예: 벡터 공간, 고유값의 변환 시각화).
   * `agent-verifier` (Quality Skill): 수식의 정확성, 용어 일관성, SVG 렌더링 가능 여부 검수.

  ---

  2. 최적의 디렉토리 구조 (Workspace Design)

  작업의 병렬성과 관리를 위해 다음과 같은 구조를 권장합니다.


   1 linear-algebra-book/
   2 ├── .agent/skills/         # 에이전트별 스킬 정의 (leader, writer, drawer, verifier)
   3 ├── src/
   4 │   ├── sections/          # writer가 생성한 텍스트 (.md)
   5 │   └── assets/            # drawer가 생성한 시각 자료 (.svg)
   6 ├── build/                 # aggregator가 병합한 중간 결과물
   7 ├── feedback/              # verifier의 검토 보고서
   8 ├── TOC.json               # leader가 관리하는 목차 데이터
   9 └── aggregate.py           # 텍스트와 이미지를 결합하는 파이썬 스크립트

  ---

  3. 단계별 워크플로우 (Execution Pipeline)


  단계 1: 설계 (Leader)
  agent-leader를 활성화하여 전체 목차와 섹션별 요구사항(텍스트 길이, 포함될 이미지 설명 등)을 TOC.json으로 추출합니다.


  단계 2: 병렬 생성 (Writer & Drawer)
  Gemini CLI에서 직접 병렬 처리를 수행하거나, 제가 쉘 스크립트를 작성하여 여러 개의 작업을 동시에 실행하도록 명령할 수
  있습니다.
   * Writer: "고유값과 고유벡터의 정의" 섹션 집필.
   * Drawer: "선형 변환에 의한 공간의 왜곡"을 시각화하는 SVG 생성.


  단계 3: 병합 (Aggregator)
  run_shell_command를 통해 aggregate.py를 실행합니다. 이 스크립트는 src/sections/의 마크다운 파일에 src/assets/의 SVG
  코드를 적절한 위치에 삽입하여 하나의 통합된 초안을 만듭니다.


  단계 4: 검수 및 피드백 (Verifier -> Leader)
  agent-verifier가 통합본을 읽고 수학적 오류나 시각적 부조화를 찾아 feedback/report_v1.md를 생성합니다.


  단계 5: 반복 및 최종 보고 (Iteration)
   1. agent-leader가 피드백을 분석하여 수정 지시사항을 하달합니다.
   2. Writer/Drawer가 해당 부분을 수정합니다 (2회 반복).
   3. 최종 결과물이 완성되면 agent-leader가 저에게(사용자) 최종 리포트를 제출합니다.


  ---

  4. Gemini CLI 활용 핵심 팁


   1. `activate_skill` 활용: 각 작업 시작 전 해당 역할에 맞는 스킬을 활성화하여 컨텍스트를 분리하세요.
       * 예: activate_skill agent-writer 실행 후 특정 챕터 집필 지시.
   2. 상태 관리 파일: TOC.json을 데이터베이스처럼 활용하여 어떤 섹션이 완성되었고 어떤 섹션이 검토 중인지 상태를 기록하게
      합니다.
   3. 검증 자동화: agent-verifier에게만 맡기지 말고, run_shell_command로 실제 pdflatex나 SVG 유효성 검사 도구를 실행하여
      기술적인 오류를 1차적으로 걸러내게 할 수 있습니다.

  시작하시겠습니까?


  이 시스템을 구축하기 위해 가장 먼저 `agent-leader` 스킬을 생성하고 목차 설계를 시작하는 것이 좋습니다. 제가
  .agent/skills/leader/SKILL.md 파일의 초안을 작성해 드릴까요?
