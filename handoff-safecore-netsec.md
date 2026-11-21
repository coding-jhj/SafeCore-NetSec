# active-memory handoff: SafeCore-NetSec 3일차 트렁크 되돌리기 완료 + 4일차(회선 이중화) 준비

**Handoff #5** · 2026-11-08 · Lineage: #1 (2026-11-07): .anchor/.gitkeep 깃허브 추적 제외, 3일차 작업계획서 작성/수정, 본사 라우터+스위치 설정 진행 중 핸드오프; #2 (2026-11-07): 본사 라우터 VLAN+SVI 전부 완료, 전 장비 호스트네임 확정, 본사-지사 WAN 연결+정적 라우팅 완료 및 ping 성공, 명령어 로그 파일 생성, 작업계획서 업데이트 섹션 추가 후 핸드오프; #3 (2026-11-07): 스위치 6개 VLAN 미입력 확인(호스트네임만 적용돼 있었음), 포트 연결 현황 파악, 라우터 쪽 업링크 포트 확인 작업 시작 중 핸드오프; #4 (2026-11-08): 스위치 6개 VLAN+텔넷+관리IP+보안(미사용 포트 shutdown) 전부 완료, 수업 과제(트렁크 재구성)를 체인 구조(RND-QA-TS-ADM-SRV)로 물리 재배선+라우터 서브인터페이스 입력(검증 전)+스위치 VTP/트렁크 명령어 수정 후 재시도 직전 상태로 핸드오프; #5 (2026-11-08, 이번 채팅): 트렁크/VTP 적용 완료·검증(캡처 8종 확보), 트렁크를 원래 설계로 전부 되돌리기 완료·검증(ping 4/4 성공), 명령어 로그 3일차_명령어_로그.md 14~16번 섹션까지 소급 반영, 4일차 범위를 "정적 라우팅 기반+회선 이중화"로 결정, 본사 라우터에 시리얼 모듈(NIM-2T) 교체 완료 확인, 회선 이중화용 IP 설정 명령어까지 전달했으나 사용자가 아직 미실행인 상태로 핸드오프

## 0. Instructions for Claude (read first)

You are continuing work from a previous chat. That chat is gone; this file is the complete context and the source of truth.

1. Read this whole file before replying.
2. Follow sections 3 (Style), 4 (Hard rules) and 5 (Corrections) in every reply, for the rest of this chat. They override your defaults.
3. Use the values in section 8 exactly. Never round, re-estimate, or "correct" them.
4. Do not suggest anything listed in section 7 (Changed / rejected) again unless the user brings it up.
5. Code word: None active. If the user types /amcodeword, start every reply with the phrase they choose (default "Yes Boss!").
6. Your first reply: at most 5 lines covering the goal, the current state, and the next step (section 11). Mention any files from section 13 that were not attached. Ask the questions in section 12 if there are any. End with "Ready to continue with <next step>?" Then wait for the user's go.

## 1. Mission
- **Goal:** SafeCore-NetSec 포트폴리오 프로젝트(Cisco Packet Tracer). 3일차(메인 VLAN+SVI 설계 + 수업 과제인 트렁크/VTP 재구성)는 완료 및 원래 설계로 되돌리기까지 끝남. 지금은 4일차: 오늘 수업에서 배운 "정적 라우팅 + 회선 이중화"를 기존 본사-지사 토폴로지에 그대로 적용하는 작업 진행 중.
- **Done looks like:** (4일차) 본사-지사 사이에 두 번째 WAN 링크(시리얼)를 구성하고, 기존 정적 라우팅 기반으로 floating static route(백업 경로)를 추가해서 기본 회선 장애 시 자동으로 백업 경로로 전환되는 걸 확인·캡처 → 4일차_작업계획서.md 작성 → 3일차_작업계획서.md "결과" 섹션도 마저 작성 → 전체 커밋/푸시.
- **Why it matters / context:** 실습 수업과 병행 진행 중인 개인 포트폴리오 repo. 사용자는 토폴로지 자체는 바꾸지 않고(바꾸는 게 부담스러움), 그 안에서 배운 개념을 최대한 깊게 적용해서 포트폴리오 수준을 끌어올리고 싶어함. "그날 배운 걸 그날 토폴로지에 적용"하는 패턴을 계속 유지 중.

## 2. About the user (as relevant to this work)
- 네트워크 수업을 들으며 동시에 포트폴리오용 Packet Tracer 프로젝트를 진행 중. CLI 명령어에 아직 익숙하지 않아, 한 줄씩 치고 결과 확인하는 방식을 선호함. 프롬프트/에러 메시지/show 명령어 결과를 캡처해서 그대로 붙여넣는 방식으로 소통함.
- 여러 장비에 걸친 반복 작업은 한 번에 몰아서(장비별로 전부 풀어서) 받는 걸 선호함.
- 회사명: SafeCore(세이프코어), 깃 사용자: coding-jhj, 이메일: ghkswnwjd@gmail.com.
- 2개월짜리 개인 포트폴리오로 생각 중이며, Cisco Packet Tracer 안에서 "최대한 깊게" 가고 싶어함. 다만 토폴로지 자체를 바꾸는 건 부담스러워해서, 기존 토폴로지를 유지한 채 설정/개념을 깊게 쌓는 방향을 선호.
- 전략/범위 판단이 필요한 질문("4일차 이 정도면 될까?", "최대 깊이가 뭐야?")을 던질 때는, Claude가 직접 판단해서 추천안을 제시하길 원함 — 질문을 다시 사용자에게 되돌리면 "그럼 네가 나한테 물으면 어떡해, 의논해야지"라고 명확히 지적함. 다만 순수 기술적 사실(포트 상태, 어느 모듈인지 등)은 추측하지 말고 show 명령어로 확인.
- 명령어를 세션 중에 바로바로 로그 파일(예: 3일차_명령어_로그.md)에 저장해주길 기대함. 자동으로 되는 줄 알고 있다가 두 번(과거 세션 + 이번 세션) 안 되고 있는 걸 지적함.

## 3. Style & communication
- **Language:** 한국어로만 답변.
- **Tone:** 쉽고 구체적으로. 바로 쓸 수 있는 명령어 우선. **항상 존댓말 사용**.
- **Reply length:** 여러 장비에 적용할 반복 명령어는 장비별로 전부 풀어서 한 번에 몰아서 제공. 생략/요약("위와 동일하게") 금지. 복잡한 작업(트렁크 재구성 등)은 전체 순서를 먼저 표/목록으로 보여준 뒤 단계별로 진행.
- **Formatting:** 마크다운 표/코드블록 적극 사용. 물결(~) 절대 금지(취소선 렌더링). 한자 사용 금지. 이모지는 요청 전까지 쓰지 않음(체크포인트 등 스킬 출력에도 이모지 대신 "주의"/"위험"으로 대체).
- **Working style:** 기술적으로 불확실한 부분은 추측하지 말고 `show` 명령어 등으로 먼저 확인시킬 것. 명령어를 줄 때는 어느 장비(호스트네임)에서 입력하는지 항상 명시하고, 전체 명령어 블록을 생략 없이 적을 것. VLAN 이름 등 "안 써도 되는" 디테일도 먼저 챙겨서 제안. 여러 장비 적용 시 기존 설정 제거 → 새 설정 순서로. 설계/전략 판단은 Claude가 먼저 추천안을 제시하고, 이유("왜 이게 나은지")를 같이 설명할 것.
- **Avoid:** AI스러운 과장된 포장 문구. 근거 없는 "요구사항" 인용. 물리적 연결/케이블/모듈 확인을 GUI 클릭/마우스 hover로만 안내하는 것(CLI로 가능하면 CLI 우선, 케이블 작업 자체는 GUI 전용이라 어쩔 수 없는 경우만 인정). 사용자가 한 작업을 검증 없이 "완료됐다"고 단정하는 것.

## 4. Hard rules (word for word)
1. "~" 쓰지 마 (취소선으로 보임) / 한자도 못 읽음
2. "오늘 안에 반드시 끝낸다"류 포장 문구 쓰지 말 것
3. 혼란스러울 때는 조각조각 말고 처음부터 끝까지 순서대로 정리해서 한 번에 줄 것
4. 토폴로지 화면 클릭 대신 항상 CLI 명령어로 확인하는 방법을 우선 제시할 것
5. 기술적으로 확신 없는 내용을 추측으로 단정해서 틀린 명령어를 주지 말 것. 불확실하면 `?`/`show` 같은 진단 명령으로 먼저 확인시킬 것.
6. 항상 존댓말 사용
7. 반복 장비 명령어를 "위와 동일" 식으로 줄이지 말고 장비별로 전부 풀어서 쓸 것
8. VLAN 이름 지정 같은 선택적/문서화용 디테일도 먼저 챙겨서 제안할 것

## 5. Corrections log
| # | Claude did | The user wanted |
|---|---|---|
| 1 | 3일차 계획서에 지어낸 "요구사항" 문구를 인용 | 실제 `01_요구사항/프로젝트_계획서.md` 원문을 확인해서 정확히 인용 |
| 2 | 목표 문단에 "오늘 안에... 완성" 식 AI스러운 포장 문구 작성 | 간결하고 직설적인 서술로 수정 |
| 3 | 2일차 /24 배정 이유를 "설정 실수 방지" 중심 서술 | "/24로 더 많은 PC(254대) 수용 가능"한 확장성 중심 서술 |
| 4 | Router-on-a-Stick 방식으로 3일차 계획 작성 | 실제 토폴로지는 부서별 전용 포트 — VLAN+SVI로 전면 수정 (ROS는 완전 폐기가 아니라 수업 과제로 부활) |
| 5 | 라우터 포트 매핑을 토폴로지 GUI 클릭으로 확인 안내 | CLI(`show cdp neighbors`)로 확인 |
| 6 | "라우터 비번 바꾸는 법"을 enable부터 설명 | 이미 로그인된 상태 기준 `configure terminal`부터 |
| 7 | 연결선 "점선→실선" 질문에 케이블 종류로 답변 | 실제 질문은 "모듈 추가로 리로드된 것"에 대한 질문이었음 |
| 8 | "no switchport"로 NIM-ES2-4 포트를 라우티드 포트화 시도 | "% Incomplete command" 에러 — VLAN+SVI 방식으로 전환 |
| 9 | "vlan 30"을 라우터 글로벌 모드에서 입력하는 명령어 제공 | "% Invalid input" — 인터페이스 모드에서 자동 생성 확인 |
| 10 | WAN 케이블 확인을 GUI 마우스 hover로 안내 | CLI(`no shutdown` 후 상태 비교)로 변경 |
| 11 | 지사 라우터 명령어를 코드블록 앞에 장비명 안 밝히고 제공 | 코드블록 앞에 항상 장비 이름 먼저 명시 |
| 12 | 핸드오프 시작 시 반말로 답변 | 이후 전부 존댓말 |
| 13 | 수업 과제(트렁크) 관련 질문을 순서 무시하고 먼저 물어봄 | 정해진 순서를 혼동하지 말 것 |
| 14 | QA/TS/ADM/SRV/TSBR 스위치 텔넷 설정을 "위와 똑같이"로 생략 | 장비별로 명령어 전체를 다 작성 |
| 15 | VLAN 이름 지정을 먼저 안내 안 하고 건너뜀 | 먼저 챙겨서 제안 |
| 16 | 라우터 서브인터페이스 설정 시 기존 IP/SVI 제거를 뒤에 배치 | "% overlaps with" 에러 발생 — 기존 설정 제거를 먼저 하도록 순서 수정 |
| 17 | 2960 스위치에 `switchport trunk encapsulation dot1q` 제공 | "% Invalid input" — 2960은 802.1Q 전용이라 명령어 자체가 없음, `switchport mode trunk`만 사용 |
| 18 | 3일차_명령어_로그.md를 세션 중 실시간 업데이트 안 함 | 사용자 확인 요청 → 소급 반영 |
| 19 | 수정된 트렁크 명령어를 일부(이어서 입력할 부분)만 조각으로 제공 | 전체를 처음부터 다시, 쉽고 구체적으로 재설명 |
| 20 | 핸드오프 재개 후 첫 답변이 너무 짧아 이해 안 됨 | 더 자세히, 맥락 포함해서 재설명 |
| 21 | VTP/트렁크 에러 수정 시 또다시 "에러 난 줄 이후부터"만 조각으로 제공(교정 #19 재발) | "C부터 다시 제대로 해야 하는데 왜 다 했다고 하냐" — C단계(RND) 전체를 처음부터 재작성 |
| 22 | 스위치 케이블을 "Auto"로 재연결한 뒤 어느 포트인지 확인 없이 되돌리기 명령어 제공 | "auto로 연결했는데 포트 알고 명령어 주는거야?" — `show cdp neighbors`/`show ip interface brief`로 실제 포트 확인부터 하도록 수정 |
| 23 | 라우터 포트가 빨간색(down)인 걸 "방금 연결해서 그런 것"으로 사용자가 추측 | 실제 원인은 라우터 인터페이스가 트렁크 작업 때 shutdown된 채 안 살아난 것이었음 — 원인 정확히 진단 |
| 24 | 명령어를 3일차_명령어_로그.md에 실시간 저장하지 않음(세션 중 재발) | "자꾸 잊는거 같은데 계속 저장하고 있지?" — 소급 반영 + 앞으로 매번 바로 저장하기로 메모리(feedback_log_commands_continuously.md)에 등록 |
| 25 | 회선 이중화 설계(정적 라우팅 유지/대체 여부)를 사용자에게 되물음 | "너랑 나랑 의논해야지, 왜 나한테 물어?" — Claude가 먼저 추천 로드맵(단계별 깊이표)을 제시하는 방식으로 전환 |

## 6. Decisions
| Decision | Why |
|---|---|
| 호스트네임: SC-HQ-R1, SC-BR-R1, SC-SW-RND/QA/TS/ADM/SRV/TSBR | 사용자 확정, 적용 완료 |
| 전 장비 비밀번호(enable secret/console/vty) 통일: `Safecore1!` | 실습용 |
| 3일차 범위에 텔넷 설정 포함 | 사용자 요청 |
| WAN 대역 192.168.70.0/24(VLAN 70) | 기존 체계 통일 |
| 명령어 로그는 `3일차_명령어_로그.md`에 계속 기록, 매번 바로바로 저장 | 사용자 요청, 두 번 재확인됨 — 메모리에도 등록 완료 |
| 수업 과제(트렁크 재구성)도 3일차 포트폴리오 과정에 포함 | 사용자 요청 |
| 트렁크 구조는 체인(RND-QA-TS-ADM-SRV)으로 진행 | 사용자가 토폴로지에서 직접 연결함 |
| VTP 도메인명 `SAFECORE`, SC-SW-RND=서버, 나머지 4개=클라이언트 | 적용·검증 완료 |
| 트렁크/VTP 완료 후 원래 설계(부서별 직결+VLAN/SVI)로 되돌리기 | **확정 및 실행 완료**, ping 4/4로 최종 검증됨 |
| 4일차 범위: "기존 정적 라우팅 기반으로 본사-지사 간 회선 이중화(백업 경로) 구성" | 동적 라우팅은 다음 수업 이후로 연기, 오늘 수업 진도(정적+회선이중화)에 맞춤 |
| 회선 이중화는 두 번째 WAN 링크를 **시리얼**로 구성 | 기본 회선(이더넷)과 다른 매체를 써야 "진짜 이중화" 개념을 더 잘 보여줌 |
| 본사 NIM 슬롯 중 미사용 "NIM 2"(Gi0/2/x)만 NIM-2T로 교체, 사용 중인 "NIM 1"(Gi0/1/x)은 보존 | 메인 설계에 영향 없이 안전하게 교체 — **실행 완료, Serial0/2/0·0/2/1 생성 확인됨** |
| 새 시리얼 링크 대역: 192.168.80.0/30 | point-to-point 구간이라 주소 2개만 필요, 부서망(/24)과 구분 |

## 7. Changed / rejected
- Router-on-a-Stick(메인 포트폴리오) → VLAN+SVI로 진행 (ROS 자체는 수업 과제로 되살아남, 메인과는 별개)
- "no switchport"로 라우티드 포트화 → 실패, VLAN+SVI로 전환
- WAN 대역 10.0.0.0/30(최초 제안) → 192.168.70.0/24
- `switchport trunk encapsulation dot1q` 사용 시도 → 2960 미지원으로 제거, `switchport mode trunk`만 사용
- 트렁크 구조를 최종 상태로 포트폴리오에 남기는 안 → 기각, 원래 설계로 되돌리기로 확정(실행 완료)
- 오늘 바로 동적 라우팅까지 하자는 생각 → 다음 수업 이후로 연기, 4일차는 정적+회선이중화로 범위 축소
- 두 번째 WAN 링크를 지사에 이더넷 모듈(HWIC-4ESW) 추가하는 안 → 기각, 본사에 시리얼 모듈(NIM-2T) 추가하는 안으로 확정(다른 매체 사용이 더 현실적인 이중화 개념이라는 이유)

## 8. Data & facts (exact)

**부서/구간별 네트워크 (전부 확정, 메인 설계 그대로 유지 중)**

| 구간 | VLAN | 대역 | 게이트웨이 |
|------|------|------|-----------|
| 연구개발팀 | 10 | 192.168.10.0/24 | 192.168.10.1 |
| 품질보증팀(QA) | 20 | 192.168.20.0/24 | 192.168.20.1 |
| 기술지원팀(본사) | 30 | 192.168.30.0/24 | 192.168.30.1 |
| 경영지원팀 | 40 | 192.168.40.0/24 | 192.168.40.1 |
| 서버팜 | 50 | 192.168.50.0/24 | 192.168.50.1 |
| 기술지원팀(지사) | 60 | 192.168.60.0/24 | 192.168.60.1 |
| WAN(본사-지사, 기본 회선) | 70 | 192.168.70.0/24 | 본사 192.168.70.1 / 지사 192.168.70.2 |
| WAN(본사-지사, **백업 회선**, 신규) | - | 192.168.80.0/30 | 본사(Serial0/2/0) 192.168.80.1 / 지사(Serial0/3/0) 192.168.80.2 |

**본사 라우터 SC-HQ-R1 — 메인 설계 인터페이스 (트렁크 되돌리기 후 현재 상태, 검증 완료)**

| 구간 | 인터페이스 | 방식 |
|------|-----------|------|
| 연구개발팀 | GigabitEthernet0/0/0 | 직접 ip address 192.168.10.1 |
| 품질보증팀 | GigabitEthernet0/0/1 | 직접 ip address 192.168.20.1 |
| 기술지원팀(본사) | GigabitEthernet0/1/0 (NIM 1) | VLAN30+SVI, ip 192.168.30.1 |
| 경영지원팀 | GigabitEthernet0/1/1 (NIM 1) | VLAN40+SVI, ip 192.168.40.1 |
| 서버팜 | GigabitEthernet0/1/2 (NIM 1) | VLAN50+SVI, ip 192.168.50.1 |
| WAN(지사, 기본) | GigabitEthernet0/1/3 (NIM 1) | VLAN70+SVI, ip 192.168.70.1 |
| WAN(지사, 백업, 신규) | Serial0/2/0 (NIM 2 자리, NIM-ES2-4→NIM-2T로 교체됨) | ip address 명령어 전달함, **사용자 미실행** |

**지사 라우터(SC-BR-R1, 2901)**
- `GigabitEthernet0/0`: 192.168.60.1/24, SC-SW-TSBR 연결
- `GigabitEthernet0/1`: WAN(기본), 192.168.70.2/24, SC-HQ-R1 Gi0/1/3과 연결
- `Serial0/3/0`: WAN(백업, 신규), ip address 명령어 전달함, **사용자 미실행**
- `Serial0/3/1`: 미사용(여유)

**정적 라우팅 (기존, 메인 설계에 계속 유지 중)**
- SC-HQ-R1: `ip route 192.168.60.0 255.255.255.0 192.168.70.2`
- SC-BR-R1: `ip route 192.168.10.0/.20.0/.30.0/.40.0/.50.0 255.255.255.0 192.168.70.1` (5개 개별 라인)

**스위치 6개 — connected 포트 범위, VLAN, 이름, 라우터 업링크 포트(메인 설계, 트렁크 되돌리기 후 재검증 완료)**

| 스위치 | connected 포트 | VLAN | VLAN 이름 | 라우터 업링크 포트 |
|---|---|---|---|---|
| SC-SW-RND | Fa0/1-15 | 10 | RnD | Fa0/1 → SC-HQ-R1 Gi0/0/0 |
| SC-SW-QA | Fa0/1-7 | 20 | QA | Fa0/1 → SC-HQ-R1 Gi0/0/1 |
| SC-SW-TS | Fa0/1-7 | 30 | TechSupport_HQ | Fa0/4 → SC-HQ-R1 Gi0/1/0 |
| SC-SW-ADM | Fa0/1-9 | 40 | Admin | Fa0/4 → SC-HQ-R1 Gi0/1/1 |
| SC-SW-SRV | Fa0/1-5 | 50 | ServerFarm | Fa0/5 → SC-HQ-R1 Gi0/1/2 |
| SC-SW-TSBR | Fa0/1-9 | 60 | TechSupport_Branch | Fa0/1 → SC-BR-R1 Gi0/0 |

안 쓰는 포트는 전부 shutdown(Fa0/16-24 등 + Gig0/1-2, 스위치마다 범위 다름). 트렁크 되돌리기 후에도 이 상태 그대로 재확인됨.

**스위치 관리용 IP (텔넷용)**

| 스위치 | 관리 IP | 게이트웨이 |
|---|---|---|
| SC-SW-RND | 192.168.10.254 | 192.168.10.1 |
| SC-SW-QA | 192.168.20.254 | 192.168.20.1 |
| SC-SW-TS | 192.168.30.254 | 192.168.30.1 |
| SC-SW-ADM | 192.168.40.254 | 192.168.40.1 |
| SC-SW-SRV | 192.168.50.254 | 192.168.50.1 |
| SC-SW-TSBR | 192.168.60.254 | 192.168.60.1 |

**비밀번호/계정**: enable secret / console / vty 전부 `Safecore1!` (실습용)

**본사 라우터 모듈 구성 (ISR4331)**
- NIM 1 슬롯: NIM-ES2-4 (사용 중, Gi0/1/0~3) — 건드리지 않음
- NIM 2 슬롯: ~~NIM-ES2-4(미사용)~~ → **NIM-2T로 교체 완료**, `Serial0/2/0`·`Serial0/2/1` 생성 확인됨(2026-11-08)

**지사 라우터 모듈 구성 (2901)**
- 온보드: GigabitEthernet0/0, 0/1 (사용 중)
- WIC 슬롯(3번): HWIC-2T로 추정, `Serial0/3/0`·`Serial0/3/1` 제공 — 교체 불필요, 기존 것 그대로 사용

**3일차 트렁크 수업 과제 — 이미 완료 및 되돌리기까지 끝난 작업 (상세 명령어는 `3일차_명령어_로그.md` 13~15번 섹션에 전부 있음)**
- 체인 구조(RND-QA-TS-ADM-SRV), 본사 Router-on-a-Stick 서브인터페이스, VTP 서버/클라이언트 구성 → 적용·검증(캡처 8종) → 전부 원래 설계로 되돌리고 재검증(ping 4/4 성공) 완료.

## 9. People, terms & names
- **People:** 사용자 = coding-jhj (개인 포트폴리오 운영자, 깃 사용자명 coding-jhj, 이메일 ghkswnwjd@gmail.com)
- **Terms & nicknames:** SafeCore(세이프코어) = 가상 회사명. 본사=판교, 지사=부산(고객지원센터, 기술지원팀 8명). "NIM 1"/"NIM 2" = 본사 라우터의 두 NIM 모듈 슬롯을 가리키는 사용자/Claude 간 약칭(공식 Cisco 용어 아님, Physical 탭 화면 라벨 기준).
- **Names in use:**
  - 저장소: github.com/coding-jhj/SafeCore-NetSec.git, 로컬 경로 C:\Network Project
  - Packet Tracer 파일: C:\Network Project\00_구축결과\SafeCoreNetSec.pkt
  - 작업계획서: C:\Network Project\02_작업 계획\2일차_작업계획서.md, 3일차_작업계획서.md (아직 "6. 결과" 섹션 미작성), **4일차_작업계획서.md (아직 파일 자체가 없음, 새로 만들어야 함)**
  - 명령어 로그: C:\Network Project\02_작업 계획\3일차_명령어_로그.md (16번 섹션까지 작성 완료, 17번부터 이어서 기록 필요 — 4일차 작업을 여기 계속 쓸지 새 파일을 만들지는 미정, 섹션 12 참고)
  - 요구사항 문서: C:\Network Project\01_요구사항\프로젝트_계획서.md
  - 이 핸드오프 파일: C:\Network Project\handoff-safecore-netsec.md
  - 호스트네임: SC-HQ-R1, SC-BR-R1, SC-SW-RND/QA/TS/ADM/SRV/TSBR
  - 모듈명: NIM-ES2-4, NIM-2T(신규 추가), HWIC-2T(지사 기존)

## 10. Work state
| Item | Status | Version / location | Notes |
|---|---|---|---|
| 본사/지사 라우터 메인 설계(VLAN+SVI) | 완료, 검증 완료 | 현재 최종 상태 | 트렁크 되돌리기 후 재검증까지 끝남 |
| 스위치 6개 VLAN+텔넷+관리IP+보안 | 완료, 저장됨 | - | - |
| 트렁크/VTP 수업 과제 (과정) | 완료, 되돌리기까지 끝남 | - | 캡처 8종은 사용자 로컬 Packet Tracer 화면에만 있고, 별도 이미지 파일로 저장됐는지는 미확인 |
| 3일차_명령어_로그.md | 1~16번 섹션 작성 완료 | C:\Network Project\02_작업 계획\3일차_명령어_로그.md | 매번 바로 이어서 기록할 것(하드 규칙급 — 두 번 지적됨) |
| 3일차_작업계획서.md "결과" 섹션 | 미작성 | - | 메인 설계 + 트렁크 과정 + 되돌리기 전부 포함해서 작성 필요 |
| 본사 라우터 NIM 모듈 교체 (NIM-2T) | **완료, 확인됨** | Serial0/2/0·0/2/1 생성 확인(2026-11-08) | - |
| 회선 이중화 — 케이블 연결(GUI) | 명령어/지시만 전달 | - | SC-HQ-R1 Serial0/2/0 ↔ SC-BR-R1 Serial0/3/0 |
| 회선 이중화 — IP 주소 설정 | 명령어 전달함, **사용자 미실행** | 192.168.80.1/30 (HQ), 192.168.80.2/30 (BR) | 다음 액션 |
| 회선 이중화 — clock rate(DCE) | 미시작 | - | `show controllers serial0/2/0`으로 DCE 쪽 확인부터 필요 |
| 회선 이중화 — floating static route | 미시작 | - | AD 값, next-hop 방식 등 미확정 |
| 회선 이중화 — IP SLA+track(심화, 선택) | 검토만 함, 확정 안 됨 | - | 할지 말지 사용자 확답 없음 |
| 4일차_작업계획서.md | 미작성(파일 없음) | - | 목표: "기존 정적 라우팅 기반으로 본사-지사 간 회선 이중화 구성" |
| 깃허브 커밋/푸시(3일차+4일차 분) | 미시작 | - | - |

## 11. Next steps
1. **Next action:** GUI에서 SC-HQ-R1 `Serial0/2/0` ↔ SC-BR-R1 `Serial0/3/0` 케이블 연결 → 양쪽에 IP 설정 입력:
   - SC-HQ-R1: `interface Serial0/2/0` → `ip address 192.168.80.1 255.255.255.252` → `no shutdown`
   - SC-BR-R1: `interface Serial0/3/0` → `ip address 192.168.80.2 255.255.255.252` → `no shutdown`
2. SC-HQ-R1에서 `show controllers serial0/2/0` 확인 → DCE/DTE 쪽 파악 → DCE 쪽에 `clock rate` 명령어 추가
3. 양쪽 `show ip interface brief`로 up/up 확인
4. floating static route 추가 (AD 값, 방식은 사용자와 논의해서 확정 필요 — 섹션 12 참고)
5. (선택) IP SLA + track으로 자동 failover 구현할지 결정 후 진행
6. 기존 회선(Gi0/1/3) 일부러 shutdown해서 failover 동작 확인 + 캡처
7. 매 단계 명령어를 그때그때 3일차_명령어_로그.md(또는 4일차 신규 로그 파일, 섹션 12 참고)에 바로 반영
8. 4일차_작업계획서.md 새로 작성
9. 3일차_작업계획서.md "6. 결과" 섹션 작성
10. 전체 커밋 + 깃허브 푸시

## 12. Open questions
- 4일차 작업을 `3일차_명령어_로그.md`에 계속 이어서 기록할지, `4일차_명령어_로그.md`를 새로 만들지 아직 안 정함.
- floating static route의 administrative distance 값이 아직 미정(예시로 150 거론됐으나 확정 안 됨).
- DCE 쪽 clock rate 속도값 미정(보통 64000을 쓰지만, `show controllers` 확인 후 정해야 함 — 추측 금지, 하드 규칙 5번).
- IP SLA + track까지 할지 여부가 "검토 중"으로만 남아있고 사용자가 명확히 "한다"고 확정하지 않음.
- 트렁크 작업 캡처 8종(서브인터페이스, CDP, 스위치 5대 VTP+트렁크, ping)이 실제로 스크린샷 파일로 사용자 로컬에 저장돼 있는지 재확인 필요 — 채팅에 붙여넣은 것만 확인됨, 파일로 보관됐는지는 모름.

## 13. Re-attach checklist
Nothing to attach. (작업 파일들은 모두 C:\Network Project 저장소 안에 있고, 새 채팅에서도 같은 로컬 경로로 접근 가능함)

---
<sub>Audit: 13/13 sections · 8 hard rules · 25 corrections · 다수 data points(부서/WAN IP·VLAN표, 인터페이스 매핑표, 신규 백업 회선 정보, 정적 라우팅, 관리IP, 모듈 구성, 비밀번호 포함) · secrets removed: none found(실습용 랩 비밀번호만 포함, 운영 비밀 아님) · generated by active-memory</sub>
